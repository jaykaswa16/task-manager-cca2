
pipeline {
    agent any

    stages {
        stage('GitHub Checkout') {
            steps {
                echo 'GitHub checkout successful!'
            }
        }

        stage('Docker Connection Test') {
            steps {
                bat '"C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" info'
            }
        }
    }
}
