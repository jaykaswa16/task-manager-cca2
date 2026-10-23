
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

        stage('Build Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Test Image') {
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
                        $configDir = Join-Path $env:WORKSPACE "docker-auth-temp"

                        New-Item -ItemType Directory -Force -Path $configDir | Out-Null
                        $env:DOCKER_CONFIG = $configDir

                        try {
                            $env:DOCKER_TOKEN.Trim() |
                                & $env:DOCKER_EXE login `
                                    --username $env:DOCKER_USER `
                                    --password-stdin

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker Hub authentication failed."
                            }
                        }
                        finally {
                            Remove-Item Env:DOCKER_CONFIG -ErrorAction SilentlyContinue
                            Remove-Item -LiteralPath $configDir -Recurse -Force -ErrorAction SilentlyContinue
                        }
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                bat '"%DOCKER_EXE%" push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                bat '"%DOCKER_EXE%" push %DOCKER_IMAGE%:latest'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built and pushed.'
        }
        failure {
            echo 'Pipeline failed. Check the first error in Console Output.'
        }
    }
}
