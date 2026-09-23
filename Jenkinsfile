pipeline {
    agent { label 'docker' }

    stages {
        stage('Checkout') {
            steps {
                echo 'RustDesk repository checked out successfully'
            }
        }

        stage('Validate') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Docker Test') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }
    }
}