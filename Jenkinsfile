pipeline {
    agent any

    tools {
        maven 'Maven-3.9.16'
    }

    environment {
        APP_NAME = 'shopping-app'
        NEW_VERSION = 'v2'
        OLD_VERSION = 'v1'
        APP_PORT = '8090'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Build Image') {
            steps {
                bat 'docker build -t %APP_NAME%:%NEW_VERSION% .'
            }
        }

        stage('Deploy New Version') {
            steps {
                script {
                    bat returnStatus: true, script: 'docker stop %APP_NAME%'
                    bat returnStatus: true, script: 'docker rm %APP_NAME%'
                }

                bat 'docker run -d --name %APP_NAME% -p %APP_PORT%:8080 %APP_NAME%:%NEW_VERSION%'
            }
        }

        stage('Health Check') {
            steps {
                sleep time: 10, unit: 'SECONDS'
                bat 'curl.exe --retry 5 --retry-delay 2 -f http://localhost:%APP_PORT%/invalid-health'
            }
        }
    }

    post {
        success {
            echo 'Version 2 deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed. Initiating rollback...'

            script {
                bat returnStatus: true, script: 'docker stop %APP_NAME%'
                bat returnStatus: true, script: 'docker rm %APP_NAME%'
            }

            bat 'docker run -d --name %APP_NAME% -p %APP_PORT%:8080 %APP_NAME%:%OLD_VERSION%'

            echo 'Rollback to version 1 completed.'
        }
    }
}