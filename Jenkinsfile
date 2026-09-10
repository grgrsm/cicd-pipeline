pipeline {
    agent any

    environment {
        CURRENT_BRANCH = "${ENV_BRANCH ? ENV_BRANCH : (BRANCH_NAME ? BRANCH_NAME : 'main')}"
        PORT = "${CURRENT_BRANCH == 'main' ? '3000' : '3001'}"
        IMAGE_NAME = "${CURRENT_BRANCH == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        CONTAINER_NAME = "${CURRENT_BRANCH == 'main' ? 'app-main' : 'app-dev'}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Docker build') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        if [ \$(docker ps -a -q -f name=^/${CONTAINER_NAME}\$) ]; then
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                        docker run -d --name ${CONTAINER_NAME} -p ${PORT}:3000 ${IMAGE_NAME}
                    """
                }
            }
        }
    }
}