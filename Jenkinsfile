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
                sh 'yarn test:unit'
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
                script {
                    try {
                        sh 'yarn test:component:ci'
                    } catch (Exception e) {
                        unstable('Component tests failed')
                    }
                }
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint= --network=host'
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        sh '''
                            // yarn db:seed:dev
                            // yarn prestart:ci
                            npm run dev
                            sleep 30
                            yarn test:headless
                        '''
                    } catch (Exception e) {
                        unstable('E2E tests failed')
                    } finally {
                        sh 'pkill -f "node" || true'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'cypress/videos/**/*.mp4,cypress/screenshots/**/*.png', allowEmptyArchive: true
                }
            }
        }
    }
}