
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'frontend-website'
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
                    test -f index.html
                    test -f style.css
                    test -f script.js
                    echo "Website files validated successfully"
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
                    sleep 3
                    curl -f http://127.0.0.1/
                    echo "Deployment successful!"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
    }
}
