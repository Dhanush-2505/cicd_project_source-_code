pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'
        PATH = "/opt/sonar-scanner/bin:${env.PATH}"

        IMAGE_NAME = 'fastapi-app'
        ECR_REPO = '429722559165.dkr.ecr.ap-southeast-1.amazonaws.com/fastapi-app'
        AWS_REGION = 'ap-southeast-1'

        DEPLOYMENT_REPO = 'https://github.com/Dhanush-2505/cicd_deployment.git'
    }

    stages {

        stage("Checkout Source Code") {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/Dhanush-2505/cicd_project_source-_code.git'
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    def shortHash = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    env.IMAGE_TAG = "${BUILD_NUMBER}-${shortHash}"
                    echo "Image Tag: ${IMAGE_TAG}"
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                    sonar-scanner \
                      -Dsonar.projectKey=fastapi-cicd \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Deployment Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'git-cred',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )]) {
                    sh '''
                        rm -rf deploy-repo

                        git clone https://$GIT_USERNAME:$GIT_PASSWORD@github.com/Dhanush-2505/cicd_deployment.git deploy-repo

                        cd deploy-repo

                        FILE=$(find . -name "deployment.yaml" | head -n 1)

                        if [ -z "$FILE" ]; then
                          echo "deployment.yaml not found ❌"
                          exit 1
                        fi

                        echo "Updating $FILE"

                        sed -i "s#image: .*#image: '"${ECR_REPO}:${IMAGE_TAG}"'#g" $FILE

                        git config user.email "jenkins@local"
                        git config user.name "Jenkins"

                        git add .
                        git commit -m "Update image ${IMAGE_TAG}" || echo "No changes"

                        git push origin main
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f deploy-repo/k8s/deployment.yaml
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS 🚀"
        }
        failure {
            echo "Pipeline FAILED ❌ — Check logs properly"
        }
    }
}