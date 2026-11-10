
pipeline {
    agent any

    environment {
        IMAGE_REPO = '3booda24/nodejs8'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'nodejs-container'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'Master',
                    url: 'https://github.com/abdelrahmanonline4/GitOps-ci-cd-with-Jenkins-and-Argocd'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Dockerhub-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker Image..."
                    docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "Pushing Docker Image..."
                    docker push ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                    echo "Removing Old Container..."
                    docker rm -f ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Deploy New Container') {
            steps {
                sh '''
                    echo "Deploying Docker Container..."

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p 3000:3000 \
                        ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Container') {
            steps {
                sh '''
                    echo "Verifying Container..."
                    docker ps --filter "name=${CONTAINER_NAME}"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline Completed Successfully!'
        }

        failure {
            echo 'CI/CD Pipeline Failed!'
        }
    }
}
