pipeline {
    agent { label 'docker' }

    stages {
        stage('Validate Staging Compose') {
            steps {
                echo 'Validating RustDesk staging configuration'
                sh 'docker compose -f compose.staging.yml config'
            }
        }
    }
}