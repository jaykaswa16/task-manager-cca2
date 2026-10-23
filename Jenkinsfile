pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jaykaswa16/task-manager'
        DOCKER_CREDENTIALS = 'dockerhub-creds-final'
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
                bat 'python --version'
                bat '"%DOCKER_EXE%" image inspect %DOCKER_IMAGE%:latest'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds-final',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    powershell '''
         if ([string]::IsNullOrWhiteSpace($env:DOCKER_USER)) {
    Write-Host "DIAGNOSTIC: Docker username is empty"
    exit 1
}

if ([string]::IsNullOrWhiteSpace($env:DOCKER_TOKEN)) {
    Write-Host "DIAGNOSTIC: Docker token is empty"
    exit 1
}

Write-Host "DIAGNOSTIC: Username is present"
Write-Host "DIAGNOSTIC: Token is present"
Write-Host "DIAGNOSTIC: Token length is $($env:DOCKER_TOKEN.Length)"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                bat '"%DOCKER_EXE%" push %DOCKER_IMAGE%:latest'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output for details.'
        }

        always {
            bat '"%DOCKER_EXE%" logout || exit /b 0'
        }
    }
}
