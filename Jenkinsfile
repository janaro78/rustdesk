pipeline {
    agent { label 'docker' }

    environment {
        COMPOSE = 'docker compose -p rustdesk-staging -f compose.staging.yml'
        VOLUME  = 'rustdesk-staging_rustdesk-staging-data'
    }

    stages {

        stage('Validate') {
            steps {
                sh '${COMPOSE} config'
            }
        }

        stage('Pull Images') {
            steps {
                sh '${COMPOSE} pull'
            }
        }

        stage('Create Staging Volume') {
            steps {
                // Creates the volume without starting RustDesk
                sh '${COMPOSE} create'
            }
        }

        stage('Install RustDesk Keys') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'id_ed25519',
                        variable: 'RUSTDESK_PRIVATE_KEY'
                    ),
                    file(
                        credentialsId: 'id_ed25519.pub',
                        variable: 'RUSTDESK_PUBLIC_KEY'
                    )
                ]) {
                    sh '''
                        docker run --rm \
                            -v ${VOLUME}:/rustdesk \
                            -v "$RUSTDESK_PRIVATE_KEY":/keys/id_ed25519:ro \
                            -v "$RUSTDESK_PUBLIC_KEY":/keys/id_ed25519.pub:ro \
                            alpine \
                            sh -c '
                                cp /keys/id_ed25519 /rustdesk/id_ed25519 &&
                                cp /keys/id_ed25519.pub /rustdesk/id_ed25519.pub &&
                                chmod 600 /rustdesk/id_ed25519 &&
                                chmod 644 /rustdesk/id_ed25519.pub
                            '
                    '''
                }
            }
        }

        stage('Deploy Staging') {
            steps {
                sh '${COMPOSE} up -d'
            }
        }

        stage('Verify') {
            steps {
                sh '${COMPOSE} ps'
            }
        }
    }
}