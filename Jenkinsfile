pipeline {

    agent {
        label 'ci-agent'
    }

    tools {
        git 'Default'
        dockerTool 'Docker'
    }

    environment {
        IMAGE_NAME = 'akash290698/cicd-app'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    branch: 'master',
                    credentialsId: '48c4a85a-a356-4846-93e8-3878472bab61',
                    url: 'https://github.com/AKASH-GIT-2998/my-cicd-project.git'
                )
            }
        }

        stage('Verify Git and Docker') {
            steps {
                sh '''
                    echo "===== Git ====="
                    which git
                    git --version

                    echo "===== Docker ====="
                    which docker
                    docker --version

                    echo "===== Docker Access ====="
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    sudo docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'b97860e0-84c6-411e-be9e-7d0a5953c213',
                        usernameVariable: 'akash290698',
                        passwordVariable: 'Akash@123.'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        sudo docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        sudo docker push ${IMAGE_NAME}:latest

                        sudo docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
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
            echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed at build #${BUILD_NUMBER}"
        }
    }
}
