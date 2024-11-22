pipeline {
    agent any

    stages {
        stage('Install') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                sh 'yarn install --frozen-lockfile'
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                sh 'yarn test:unit:ci'  // Using CI version of unit tests
            }
        }

        stage('Component Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                sh 'yarn test:component:ci'  // Using CI version of component tests
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # Setup and start the application in CI mode
                    yarn prestart:ci
                    yarn start:ci &
                    
                    # Wait for services
                    sleep 45
                    
                    # Run E2E tests headless
                    yarn test:headless
                '''
            }
            post {
                always {
                    sh 'pkill -f "node" || true'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'cypress/videos/**/*.mp4,cypress/screenshots/**/*.png', allowEmptyArchive: true
        }
    }
}