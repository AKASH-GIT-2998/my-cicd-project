```groovy
pipeline {

    agent {
        label 'ci-agent'
    }

    environment {
        IMAGE_NAME = 'akash290698/cicd-app'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    echo "Checking tools..."
                    git --version
                    docker --version
                    docker info
                    kubectl version --client
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "Logging in to Docker Hub..."

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "Pushing Docker image..."

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying application to Kubernetes..."

                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl rollout status deployment/cicd-app
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            echo "Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed at build #${BUILD_NUMBER}"
        }

        always {
            sh '''
                docker image prune -f || true
            '''
        }
    }
}
```
