pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME = "amoghk22/campusfit-app"
        DEPLOYMENT_NAME = "campusfit"
        CONTAINER_NAME = "campusfit"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}:${BUILD_NUMBER}"

                bat """
                    docker build -t %IMAGE_NAME%:${BUILD_NUMBER} .
                    docker tag %IMAGE_NAME%:${BUILD_NUMBER} %IMAGE_NAME%:latest
                """
            }
        }

        stage('Push Image') {
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

        stage('Debug Kubernetes') {
            steps {
                bat """
                    kubectl version --client
                    kubectl config current-context
                    kubectl cluster-info
                    kubectl get deployments
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