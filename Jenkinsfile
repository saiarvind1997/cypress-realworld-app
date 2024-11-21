pipeline {
    agent {
        docker {
            image 'cypress/included:13.16.0'
            args '-v $WORKSPACE:/e2e -w /e2e --entrypoint='  // Added --entrypoint= here
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'yarn install --frozen-lockfile'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'yarn test:unit:headless'
            }
        }
        stage('Component Tests') {
            steps {
                sh 'CYPRESS_CRASH_REPORTS=0 yarn test:component:headless'
            }
        }
        stage('E2E Tests') {
            steps {
                sh '''
                    yarn db:seed:dev
                    yarn start:ci &
                    APP_PID=$!
                    for i in $(seq 1 30); do
                        if curl -s http://localhost:3000 > /dev/null; then
                            break
                        fi
                        if [ $i -eq 30 ]; then
                            echo "Application failed to start"
                            exit 1
                        fi
                        sleep 1
                    done
                    CYPRESS_CRASH_REPORTS=0 yarn test:e2e:headless
                    TEST_EXIT_CODE=$?
                    kill $APP_PID || true
                    exit $TEST_EXIT_CODE
                '''
            }
        }
    }
    post {
        always {
            junit 'coverage/junit/**/*.xml'
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
            sh 'pkill -f "yarn start:ci" || true'
            cleanWs()
        }
        failure {
            archiveArtifacts artifacts: 'cypress/videos/**, cypress/screenshots/**', allowEmptyArchive: true
            sh 'mkdir -p logs && yarn logs || true'
            archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
        }
    }
}