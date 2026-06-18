pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker image: amoghk22/campusfit-app:${BUILD_NUMBER}"

                bat """
                    docker build -t amoghk22/campusfit-app:${BUILD_NUMBER} .
                    docker tag amoghk22/campusfit-app:${BUILD_NUMBER} amoghk22/campusfit-app:latest
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

                        docker push amoghk22/campusfit-app:${BUILD_NUMBER}
                        docker push amoghk22/campusfit-app:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                    kubectl set image deployment/campusfit campusfit=amoghk22/campusfit-app:${BUILD_NUMBER}
                    kubectl rollout status deployment/campusfit
                """
            }
        }

    }
}