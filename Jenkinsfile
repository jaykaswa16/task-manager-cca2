
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jaykaswa16/task-manager'
        DOCKER_EXE = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out successfully.'
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

        stage('Test Docker Image') {
            steps {
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
                        $ErrorActionPreference = "Stop"
                        $token = $env:DOCKER_TOKEN.Trim()

                        if ([string]::IsNullOrWhiteSpace($token)) {
                            throw "Docker Hub token is empty."
                        }

                        $token | & $env:DOCKER_EXE login `
                            --username $env:DOCKER_USER `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed."
                        }
                    '''
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
            echo 'SUCCESS: Image built and pushed to Docker Hub.'
        }
        failure {
            echo 'Pipeline failed. Check the first error in Console Output.'
        }
    }
}
