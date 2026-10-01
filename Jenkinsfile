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

        /*
         * ============================================================
         * DEPLOY PIPELINE
         * ============================================================
         */

        stage('Validate') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Validating Docker Compose configuration...'
                sh '${COMPOSE} config'
            }
        }

        stage('Pull Images') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Pulling RustDesk images...'
                sh '${COMPOSE} pull'
            }
        }

        stage('Create Staging Containers') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Creating staging containers...'

                /*
                 * Create containers without starting RustDesk.
                 * This allows Jenkins to install the required
                 * RustDesk keys before the services start.
                 */
                sh '${COMPOSE} create'
            }
        }

        stage('Install RustDesk Keys') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Installing RustDesk staging keys...'

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
                echo 'Starting RustDesk staging environment...'
                sh '${COMPOSE} up -d'
            }
        }

        stage('Verify Containers') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Checking staging container status...'
                sh '${COMPOSE} ps'
            }
        }

        /*
         * ============================================================
         * TESTING
         * ============================================================
         */

        stage('Smoke Test - Ports') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                echo 'Testing RustDesk staging TCP ports...'

                sh '''
                    docker run --rm --network host alpine sh -c '
                        # Terminal fails if any command fails, so we use "set -e" to exit on error
                        set -e

                        apk add --no-cache netcat-openbsd >/dev/null

                        echo "Testing HBBS TCP 22115..."
                        nc -z -w 5 127.0.0.1 22115

                        echo "PASS: HBBS TCP 22115"

                        echo "Testing HBBS TCP 22116..."
                        nc -z -w 5 127.0.0.1 22116

                        echo "PASS: HBBS TCP 22116"

                        echo "Testing HBBR TCP 22117..."
                        nc -z -w 5 127.0.0.1 22117

                        echo "PASS: HBBR TCP 22117"

                        echo "Testing HBBR TCP 22119..."
                        nc -z -w 5 127.0.0.1 22119

                        echo "PASS: HBBR TCP 22119"

                        echo "All RustDesk staging TCP port tests passed."
                    '
                '''
            }
        }

        /*
         * ============================================================
         * DESTROY PIPELINE
         * ============================================================
         */

        stage('Destroy Staging') {
            when {
                expression { params.ACTION == 'DESTROY' }
            }
            steps {
                echo 'Destroying RustDesk staging environment...'

                sh '''
                    ${COMPOSE} down

                    echo "Removing /data-staging..."

                    docker run --rm \
                        -v /:/host \
                        alpine \
                        rm -rf /host/data-staging
                '''
            }
        }
    }

    /*
     * ============================================================
     * PIPELINE RESULT
     * ============================================================
     */

    post {

        success {
            echo 'RustDesk pipeline completed successfully.'
        }

        failure {
            echo 'RustDesk pipeline FAILED. Check the failed stage above.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}