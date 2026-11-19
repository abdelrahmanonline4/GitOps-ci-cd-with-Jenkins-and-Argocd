
pipeline {
    agent any

    environment {
        IMAGE_REPO = '3booda24/nodejs8'
        IMAGE_TAG = 'latest'
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
                    docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Approval Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        message: 'Approve Deployment to Kubernetes?',
                        ok: 'Deploy Now',
                        submitter: 'admin'
                    )
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    cd kubernetes
                    kubectl apply -f .
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

        aborted {
            echo 'Deployment Cancelled or Approval Timed Out!'
        }
    }
}
