
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jaykaswa16/task-manager'
        DOCKER_EXE = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        EXPECTED_TOKEN_FINGERPRINT = '7CB7C4547CF2653590D7A9ACE60CC623D25148ADFBC88A89AEB0EF88DA7839BA'
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

                        $tokenText = $env:DOCKER_TOKEN.Trim()
                        $sha = [Security.Cryptography.SHA256]::Create()

                        try {
                            $bytes = [Text.Encoding]::UTF8.GetBytes($tokenText)
                            $fingerprint = [BitConverter]::ToString(
                                $sha.ComputeHash($bytes)
                            ).Replace("-", "")

                            if ($fingerprint -eq $env:EXPECTED_TOKEN_FINGERPRINT) {
                                Write-Host "Credential diagnostic: MATCH"
                            } else {
                                Write-Host "Credential diagnostic: MISMATCH"
                            }
                        }
                        finally {
                            $sha.Dispose()
                            $tokenText = $null
                            $bytes = $null
                            $fingerprint = $null
                        }

                        $configDir = Join-Path $env:WORKSPACE "docker-auth-temp"
                        New-Item -ItemType Directory -Force -Path $configDir | Out-Null

                        try {
                            $env:DOCKER_CONFIG = $configDir

                            $env:DOCKER_TOKEN.Trim() |
                                & $env:DOCKER_EXE login `
                                    --username $env:DOCKER_USER `
                                    --password-stdin

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker Hub authentication failed."
                            }

                            Write-Host "Docker Hub login successful."
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
