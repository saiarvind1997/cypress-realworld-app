pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        YARN_VERSION = '1.22.19'
        NODE_PATH = "${WORKSPACE}/node-v${NODE_VERSION}-linux-x64/bin"
        YARN_PATH = "${WORKSPACE}/yarn-v${YARN_VERSION}/bin"
        PATH = "${NODE_PATH}:${YARN_PATH}:${env.PATH}"
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    // Clear existing installations if they exist
                    sh 'rm -rf node-* yarn-* yarn.tar.gz'
                    
                    sh '''
                        set -e  # Exit on any error
                        
                        echo "Installing Node.js ${NODE_VERSION}..."
                        curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz | tar xz
                        
                        echo "Installing Yarn ${YARN_VERSION}..."
                        curl -fsSL https://github.com/yarnpkg/yarn/releases/download/v${YARN_VERSION}/yarn-v${YARN_VERSION}.tar.gz | tar xz
                        
                        echo "Verifying installations..."
                        ${NODE_PATH}/node --version
                        ${NODE_PATH}/npm --version
                        ${YARN_PATH}/yarn --version
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        set -e  # Exit on any error
                        
                        echo "Installing project dependencies..."
                        yarn install --frozen-lockfile
                        
                        echo "Seeding database..."
                        yarn db:seed:dev
                    '''
                }
            }
        }

        stage('Start App and Test') {
            steps {
                script {
                    sh '''
                        set -e  # Exit on any error
                        
                        echo "Starting application in CI mode..."
                        yarn start:ci &
                        
                        echo "Waiting for application to start..."
                        for i in $(seq 1 30); do
                            if curl -s http://localhost:3000 > /dev/null; then
                                echo "Application is ready!"
                                break
                            fi
                            if [ $i -eq 30 ]; then
                                echo "Application failed to start within 30 seconds"
                                exit 1
                            fi
                            echo "Waiting... ($i/30)"
                            sleep 1
                        done
                        
                        echo "Running tests..."
                        yarn test:headless
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh '''
                    echo "Cleaning up processes..."
                    pkill -f "yarn start:ci" || true
                    
                    echo "Cleaning up workspace..."
                '''
                cleanWs()
            }
        }
        failure {
            script {
                sh '''
                    echo "Build failed. Collecting logs..."
                    mkdir -p logs
                    yarn logs || true
                '''
                archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
            }
        }
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        ansiColor('xterm')
        disableConcurrentBuilds()
    }
}