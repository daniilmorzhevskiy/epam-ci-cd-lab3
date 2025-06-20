pipeline {
    agent any

    environment {
        IMAGE_NAME = 'daniilsd/noddev'
        IMAGE_TAG = 'v1.0'
        CONTAINER_NAME = 'nodedev'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build script...'
                sh 'chmod +x scripts/build.sh && ./scripts/build.sh'
            }
        }

        stage('Test') {
            steps {
                echo 'Running test script...'
                sh 'chmod +x scripts/test.sh && ./scripts/test.sh'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Stop Existing Container') {
            steps {
                echo 'Stopping previous container if running...'
                sh """
                    docker ps -q --filter name=$CONTAINER_NAME | xargs -r docker stop
                    docker ps -aq --filter name=$CONTAINER_NAME | xargs -r docker rm
                """
            }
        }

        stage('Deploy New Container') {
            steps {
                echo 'Running new container...'
                sh """
                    docker run -d \
                      --name $CONTAINER_NAME \
                      -p 3000:3000 \
                      $IMAGE_NAME:$IMAGE_TAG
                """
            }
        }
    }
}