pipeline {
    agent any

    environment {
        // Default values – overridden dynamically
        IMAGE_NAME = ''
        APP_PORT = ''
        CONTAINER_NAME = ''
    }

    tools {
        nodejs 'node-7.8.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Branch: ${env.BRANCH_NAME}"
                    if (env.BRANCH_NAME == 'main') {
                        IMAGE_NAME = 'nodemain:v1.0'
                        APP_PORT = '3000'
                        CONTAINER_NAME = 'node_main_app'
                    } else if (env.BRANCH_NAME == 'dev') {
                        IMAGE_NAME = 'nodedev:v1.0'
                        APP_PORT = '3001'
                        CONTAINER_NAME = 'node_dev_app'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'  // You may remove `|| true` if you want test failures to break the build
            }
        }

        stage('Lint Dockerfile (Hadolint)') {
            steps {
                sh 'hadolint Dockerfile || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Clean Up Old Containers') {
            steps {
                script {
                    sh """
                        RUNNING=\$(docker ps -q --filter "name=${CONTAINER_NAME}")
                        if [ ! -z "\$RUNNING" ]; then
                          docker stop \$RUNNING
                          docker rm \$RUNNING
                        fi
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh "docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}"
            }
        }

        stage('Scan Image (Trivy)') {
            steps {
                sh "trivy image ${IMAGE_NAME} || true"
            }
        }
    }
}