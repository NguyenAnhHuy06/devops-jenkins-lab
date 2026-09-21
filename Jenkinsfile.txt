pipeline {

    agent any

    environment {
        APP_NAME = 'techzen-app'
        DOCKER_PORT = '8081'
    }

    stages {

        stage('1. Checkout Source Code') {
            steps {
                echo '=== Checkout source code ==='
                checkout scm
            }
        }

        stage('2. Build Docker Image') {
            steps {
                echo '=== Build Docker Image ==='
                sh 'docker build -t ${APP_NAME}:latest .'
            }
        }

        stage('3. Deploy Container') {
            steps {
                echo '=== Deploy Container ==='
                sh '''
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true

                    docker run -d \
                        --name ${APP_NAME} \
                        -p ${DOCKER_PORT}:80 \
                        --restart always \
                        ${APP_NAME}:latest
                '''
            }
        }

        stage('4. Health Check') {
            steps {
                echo '=== Health Check ==='
                sh 'curl -f http://localhost:${DOCKER_PORT}'
            }
        }
    }

    post {
        success {
            echo '[SUCCESS] Pipeline completed successfully.'
        }

        failure {
            echo '[FAILED] Pipeline failed.'
        }
    }
}