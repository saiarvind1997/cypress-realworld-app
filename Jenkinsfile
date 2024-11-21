pipeline {
    agent {
        docker {
            image 'cypress/included:13.16.0'
            args '-v $WORKSPACE:/e2e -w /e2e --entrypoint='
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
                sh 'yarn test:unit:ci'  // Changed to use the correct CI command
            }
        }
        stage('Component Tests') {
            steps {
                sh 'CYPRESS_CRASH_REPORTS=0 yarn test:component:ci'  // Changed to use the correct component test command
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
                    CYPRESS_CRASH_REPORTS=0 yarn test:headless  // Changed to use the correct E2E test command
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
            sh 'pkill -f "yarn start:ci" || true'
            cleanWs()
        }
        failure {
            archiveArtifacts artifacts: 'cypress/videos/**, cypress/screenshots/**', allowEmptyArchive: true
            sh '''
                mkdir -p logs
                ps aux > logs/processes.log
                free -h > logs/memory.log
                df -h > logs/disk.log
            '''
            archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
        }
    }
}