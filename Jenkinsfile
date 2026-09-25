pipeline {
    agent { label 'docker' }

    stages {
        stage('Validate Staging Compose') {
            steps {
                echo 'Validating RustDesk staging configuration'
                sh 'docker compose -p rustdesk-staging -f compose.staging.yml up -d'
            }
        }
    }
}