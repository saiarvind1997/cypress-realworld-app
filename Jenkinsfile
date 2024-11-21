pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        YARN_VERSION = '1.22.19'
        NODE_PATH = "${WORKSPACE}/node-v${NODE_VERSION}-linux-x64/bin"
        YARN_PATH = "${WORKSPACE}/yarn-v${YARN_VERSION}/bin"
        PATH = "${NODE_PATH}:${YARN_PATH}:${env.PATH}"
        DISPLAY = ':99'
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        set -e  # Exit on any error
                        
                        # Install Xvfb and related dependencies
                        sudo apt-get update
                        sudo apt-get install -y xvfb libgtk2.0-0 libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2
                        
                        # Clean previous installations
                        rm -rf node-* yarn-* yarn.tar.gz
                        
                        # Install Node.js
                        curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz | tar xz
                        
                        # Install Yarn
                        curl -fsSL https://github.com/yarnpkg/yarn/releases/download/v${YARN_VERSION}/yarn-v${YARN_VERSION}.tar.gz | tar xz
                        
                        # Verify installations
                        if ! ${NODE_PATH}/node --version || ! ${YARN_PATH}/yarn --version; then
                            echo "Failed to install Node.js or Yarn"
                            exit 1
                        fi
                        
                        # Start Xvfb
                        Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
                        
                        # Wait for Xvfb to start
                        sleep 3
                    '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        set -e
                        
                        yarn install --frozen-lockfile
                        
                        if [ ! -d "node_modules" ]; then
                            echo "Dependencies installation failed"
                            exit 1
                        fi
                        
                        yarn db:seed:dev
                    '''
                }
            }
        }
        
        stage('Start App and Test') {
            steps {
                script {
                    sh '''
                        set -e
                        
                        # Start the application
                        yarn start:ci &
                        APP_PID=$!
                        
                        # Wait for app to start (max 30 seconds)
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
                        
                        # Run tests with Xvfb
                        yarn test:headless
                        TEST_EXIT_CODE=$?
                        
                        # Kill the app and exit with test status
                        kill $APP_PID || true
                        exit $TEST_EXIT_CODE
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh '''
                    pkill -f "yarn start:ci" || true
                    pkill Xvfb || true
                '''
                cleanWs()
            }
        }
        failure {
            script {
                sh '''
                    mkdir -p logs
                    yarn logs || true
                '''
                archiveArtifacts artifacts: 'logs/**', allowEmptyArchive: true
            }
        }
    }
}