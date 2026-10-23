
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jaykaswa16/task-manager'
        DOCKER_EXE = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
    }

    stages {
        stage('Build and Test Image') {
            steps {
                bat '"%DOCKER_EXE%" info'
                bat '"%DOCKER_EXE%" build -t %DOCKER_IMAGE%:latest .'
                bat '"%DOCKER_EXE%" image inspect %DOCKER_IMAGE%:latest'
            }
        }

        stage('Compare Token Fingerprint') {
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
                        $sha = [Security.Cryptography.SHA256]::Create()
                        $bytes = [Text.Encoding]::UTF8.GetBytes($token)
                        $hashBytes = $sha.ComputeHash($bytes)
                        $hash = [BitConverter]::ToString($hashBytes).Replace("-", "")

                        Write-Output "Username: $env:DOCKER_USER"
                        Write-Output "Jenkins token fingerprint: $hash"

                        $sha.Dispose()
                        $token = $null
                        $bytes = $null
                        $hashBytes = $null
                    '''
                }
            }
        }
    }
}
