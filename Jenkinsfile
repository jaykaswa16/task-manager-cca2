
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
                bat 'docker info'
            }
        }
    }
}
