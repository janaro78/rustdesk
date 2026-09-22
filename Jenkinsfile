pipeline {
    agent any

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
    }
}