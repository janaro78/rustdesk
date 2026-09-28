pipeline {
    agent { label 'docker' }

    environment {
        COMPOSE = 'docker compose -p rustdesk-staging -f compose.staging.yml'
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

        stage('Create Staging Containers') {
            steps {
                // Create the containers but do not start them yet.
                // This ensures RustDesk does not generate its own keys
                // before Jenkins installs the required keys.
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
                        cat "$RUSTDESK_PRIVATE_KEY" | \
                            docker run --rm -i \
                            -v /data-staging:/root \
                            alpine \
                            sh -c 'cat > /root/id_ed25519 && chmod 600 /root/id_ed25519'

                        cat "$RUSTDESK_PUBLIC_KEY" | \
                            docker run --rm -i \
                            -v /data-staging:/root \
                            alpine \
                            sh -c 'cat > /root/id_ed25519.pub && chmod 644 /root/id_ed25519.pub'
                    '''
                }
            }
        }

        stage('Deploy Staging') {
            steps {
                sh '${COMPOSE} up -d'
            }
        }

        stage('Verify Staging') {
            steps {
                sh '${COMPOSE} ps'
            }
        }

    }
}