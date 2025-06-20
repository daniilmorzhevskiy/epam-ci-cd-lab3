pipeline {
    agent any

    environment {
        IMAGE_TAG = "daniilsd/nodedev:v1.0"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build script...'
                docker.image('node:18').inside {
                    sh 'chmod +x scripts/build.sh'
                    sh './scripts/build.sh'
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running test script...'
                docker.image('node:18').inside {
                    sh 'chmod +x scripts/test.sh'
                    sh './scripts/test.sh'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $IMAGE_TAG .'
            }
        }

        stage('Push Image') {
            steps {
                echo 'Pushing Docker image...'
                withDockerRegistry(credentialsId: 'dockerhub', url: '') {
                    sh 'docker push $IMAGE_TAG'
                }
            }
        }

        stage('Stop Existing Container') {
            steps {
                echo 'Stopping existing container...'
                sh 'docker ps -q --filter name=nodedev | xargs -r docker stop'
                sh 'docker ps -a -q --filter name=nodedev | xargs -r docker rm'
            }
        }

        stage('Deploy New Container') {
            steps {
                echo 'Deploying new container...'
                sh 'docker run -d --name nodedev -p 3001:3000 $IMAGE_TAG'
            }
        }
    }
}