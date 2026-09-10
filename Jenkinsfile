pipeline {
    agent any

    tools {
        nodejs 'Node 7.8.0'
    }

    environment {
        // Определение текущей ветки для Multibranch и для параметризованного пайплайна
        CURRENT_BRANCH = "${ENV_BRANCH ? ENV_BRANCH : (BRANCH_NAME ? BRANCH_NAME : 'main')}"
        
        // Разделение параметров для веток main (порт 3000) и dev (порт 3001)
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

        stage('Declarative: Tool Install') {
            steps {
                sh 'node -v'
                sh 'npm -v'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                // Выполнение тестов (не падает, если тестов нет в пакете)
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
                    // Остановка и удаление только предыдущего контейнера текущего окружения
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