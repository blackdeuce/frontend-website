
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '882638355979'
        ECR_REPO = 'frontend-website'
        ECR_REGISTRY = '882638355979.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME = '882638355979.dkr.ecr.us-east-1.amazonaws.com/frontend-website'
        CONTAINER_NAME = 'frontend-website'
        HOST_PORT = '80'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    set -e
                    test -f index.html
                    test -f style.css
                    test -f script.js
                    test -f Dockerfile
                    echo "All website files validated!"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                    -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Pull Image from ECR') {
            steps {
                sh '''
                    docker pull ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    --restart unless-stopped \
                    -p ${HOST_PORT}:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -e

                    for i in $(seq 1 10); do
                        if curl -fsS http://127.0.0.1/; then
                            echo "Website deployment successful!"
                            exit 0
                        fi
                        sleep 3
                    done

                    echo "Website health check failed!"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image pushed to ECR and website deployed!'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output.'
        }
    }
}
