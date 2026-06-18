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

    }
}