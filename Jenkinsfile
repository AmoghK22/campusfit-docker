pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME      = "amoghk22/campusfit-app"
        DEPLOYMENT_NAME = "campusfit"
        CONTAINER_NAME  = "campusfit"

        // Kubernetes config for Jenkins (LocalSystem)
        KUBECONFIG = "C:\\Windows\\System32\\config\\systemprofile\\.kube\\config"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}:${BUILD_NUMBER}"

                bat """
                    docker build -t %IMAGE_NAME%:${BUILD_NUMBER} .
                    docker tag %IMAGE_NAME%:${BUILD_NUMBER} %IMAGE_NAME%:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        docker push %IMAGE_NAME%:${BUILD_NUMBER}
                        docker push %IMAGE_NAME%:latest

                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                    kubectl set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%IMAGE_NAME%:${BUILD_NUMBER}

                    kubectl rollout status deployment/%DEPLOYMENT_NAME%
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                bat """
                    kubectl get deployment %DEPLOYMENT_NAME%
                    kubectl get pods -l app=%DEPLOYMENT_NAME%
                    kubectl get services
                """
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Check the console output for details."
        }

        always {
            cleanWs()
        }
    }
}