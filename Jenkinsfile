pipeline {
    agent { label 'ci-agent' }

    environment {
        IMAGE_NAME = "yourname/cicd-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/AKASH-GIT-2998/my-cicd-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'b97860e0-84c6-411e-be9e-7d0a5953c213',
                    usernameVariable: 'akash290698',
                    passwordVariable: 'Akash@123.'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ubuntu@<K8S_NODE_IP> '
                    kubectl set image deployment/cicd-app cicd-app=${IMAGE_NAME}:${IMAGE_TAG}
                    kubectl rollout status deployment/cicd-app
                '
                """
            }
        }
    }

    post {
        success {
            echo "PRT - CI/CD Completed Successfully - Build #${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed at build #${BUILD_NUMBER}"
        }
    }
}
