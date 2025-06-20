pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev'], description: 'Select environment to deploy')
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

        stage('Install & Build') {
            steps {
                script {
                    docker.image('node:7.8.0').inside {
                        sh 'chmod +x scripts/build.sh'
                        sh './scripts/build.sh'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image('node:7.8.0').inside {
                        sh 'chmod +x scripts/test.sh'
                        sh './scripts/test.sh'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = 'nodedev'
                    sh "docker build -t daniilsd/${imageName}:${params.IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Login & Push Image') {
            steps {
                script {
                    sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push daniilsd/nodedev:${params.IMAGE_TAG}
                    """
                }
            }
        }

        stage('Stop Existing Container') {
            steps {
                script {
                    def containerName = 'nodedev'
                    sh "docker ps -q --filter name=${containerName} | xargs -r docker stop"
                    sh "docker ps -a -q --filter name=${containerName} | xargs -r docker rm"
                }
            }
        }

        stage('Deploy New Container') {
            steps {
                script {
                    def containerName = 'nodedev'
                    def port = '3001'
                    sh "docker run -d --name ${containerName} --expose 3000 -p ${port}:3000 daniilsd/${containerName}:${params.IMAGE_TAG}"
                }
            }
        }
    }
}