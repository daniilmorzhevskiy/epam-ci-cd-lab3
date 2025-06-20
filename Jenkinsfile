pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['main', 'dev'], description: 'Select environment to deploy')
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker image tag to deploy')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = params.ENVIRONMENT == 'main' ? 'nodemain' : 'nodedev'
                    sh "docker build -t daniilsd/${imageName}:${params.IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Login & Push Image') {
            steps {
                script {
                    sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push daniilsd/${params.ENVIRONMENT == 'main' ? 'nodemain' : 'nodedev'}:${params.IMAGE_TAG}
                    """
                }
            }
        }

        stage('Stop existing container') {
            steps {
                script {
                    def containerName = params.ENVIRONMENT == 'main' ? 'nodemain' : 'nodedev'
                    sh "docker ps -q --filter name=${containerName} | xargs -r docker stop"
                    sh "docker ps -a -q --filter name=${containerName} | xargs -r docker rm"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName = params.ENVIRONMENT == 'main' ? 'nodemain' : 'nodedev'
                    def port = params.ENVIRONMENT == 'main' ? '3000' : '3001'
                    sh "docker run -d --name ${containerName} --expose 3000 -p ${port}:3000 daniilsd/${containerName}:${params.IMAGE_TAG}"
                }
            }
        }
    }
}
