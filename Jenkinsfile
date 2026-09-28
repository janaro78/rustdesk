pipeline {
    agent { label 'docker' }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['DEPLOY', 'DESTROY'],
            description: 'Choose RustDesk staging action'
        )
    }

    environment {
        COMPOSE = 'docker compose -p rustdesk-staging -f compose.staging.yml'
    }

    stages {

        stage('Validate') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '${COMPOSE} config'
            }
        }

        stage('Pull Images') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '${COMPOSE} pull'
            }
        }

        stage('Create Staging Containers') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                // Create containers without starting RustDesk.
                // Keys will be installed before RustDesk starts.
                sh '${COMPOSE} create'
            }
        }

        stage('Install RustDesk Keys') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
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
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '${COMPOSE} up -d'
            }
        }

        stage('Verify Staging') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh '${COMPOSE} ps'
            }
        }

        stage('Destroy Staging') {
            when {
                expression { params.ACTION == 'DESTROY' }
            }
            steps {
                sh '''
                    ${COMPOSE} down

                    docker run --rm \
                        -v /:/host \
                        alpine \
                        rm -rf /host/data-staging
                '''
            }
        }

    }
}