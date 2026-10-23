
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jaykaswa16/task-manager'
        DOCKER_EXE = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
    }

    stages {
        stage('GitHub Checkout') {
            steps {
                echo 'Source code checked out successfully!'
            }
        }

        stage('Docker Connection Test') {
            steps {
                bat '"%DOCKER_EXE%" info'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Test Application') {
            steps {
                bat '"%DOCKER_EXE%" image inspect %DOCKER_IMAGE%:latest'
            }
        }

        stage('Check Docker Hub Credentials') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds-final',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"

                        Write-Output "=== CREDENTIAL DIAGNOSTIC ==="

                        if ([string]::IsNullOrWhiteSpace($env:DOCKER_USER)) {
                            throw "Username was not received by Jenkins."
                        }

                        Write-Output "Username received: $env:DOCKER_USER"

                        if ([string]::IsNullOrWhiteSpace($env:DOCKER_TOKEN)) {
                            throw "Token was not received by Jenkins."
                        }

                        Write-Output "Token received: YES"
                        Write-Output "Token length: $($env:DOCKER_TOKEN.Trim().Length)"
                        Write-Output "Diagnostic complete. No login attempted."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and credential diagnostic completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
