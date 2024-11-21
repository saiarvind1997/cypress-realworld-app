pipeline {
    agent any
    
    tools {
        nodejs 'Node'  // Make sure to configure Node.js in Jenkins Global Tool Configuration
    }
    
    environment {
        NODE_ENV = 'development'
        CI = 'true'
        // Specify Node.js version
        NODE_VERSION = '18.17.0'  // Adjust version as needed
    }
    
    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    try {
                        // Check if nvm (Node Version Manager) is installed
                        sh '''
                            if ! command -v nvm &> /dev/null; then
                                echo "Installing NVM..."
                                curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                                
                                # Load NVM
                                export NVM_DIR="$HOME/.nvm"
                                [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
                            fi
                            
                            # Install and use specific Node version
                            nvm install ${NODE_VERSION}
                            nvm use ${NODE_VERSION}
                            
                            # Verify Node.js installation
                            node --version
                            npm --version
                        '''
                    } catch (Exception e) {
                        // Alternative installation method if NVM fails
                        sh '''
                            echo "NVM installation failed, trying direct NodeJS installation..."
                            if ! command -v node &> /dev/null; then
                                curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
                                sudo apt-get install -y nodejs
                                
                                # Verify installation
                                node --version
                                npm --version
                            fi
                        '''
                    }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    try {
                        sh '''
                            # Clear npm cache if needed
                            npm cache clean --force
                            
                            # Install dependencies with detailed error logging
                            npm install --verbose
                            
                            # Verify installation
                            npm list --depth=0
                        '''
                    } catch (Exception e) {
                        echo "Error during dependency installation: ${e.message}"
                        error "Failed to install dependencies. Check npm-debug.log for details."
                    }
                }
            }
        }
        
        stage('Run Development Server') {
            steps {
                script {
                    try {
                        sh '''
                            # Check if port is already in use
                            npx kill-port 3000 || true
                            
                            # Start the dev server with error logging
                            nohup npm run dev > dev-server.log 2>&1 &
                            echo $! > .dev-server.pid
                            
                            # Monitor server startup
                            attempt=1
                            max_attempts=10
                            while ! grep -q "ready" dev-server.log && [ $attempt -le $max_attempts ]; do
                                echo "Waiting for server to start (Attempt $attempt/$max_attempts)..."
                                sleep 5
                                attempt=$((attempt + 1))
                            done
                            
                            if [ $attempt -gt $max_attempts ]; then
                                echo "Server failed to start. Server logs:"
                                cat dev-server.log
                                exit 1
                            fi
                        '''
                    } catch (Exception e) {
                        echo "Error starting development server: ${e.message}"
                        sh 'cat dev-server.log || true'
                        error "Failed to start development server"
                    }
                }
            }
        }
        
        stage('Run Headless Tests') {
            steps {
                script {
                    try {
                        sh 'npm run test:headless'
                    } catch (Exception e) {
                        echo "Error during test execution: ${e.message}"
                        error "Tests failed. Check test reports for details."
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup: Stop the dev server if it's running
                sh '''
                    if [ -f .dev-server.pid ]; then
                        pid=$(cat .dev-server.pid)
                        kill $pid || true
                        rm .dev-server.pid
                    fi
                    
                    # Archive logs
                    if [ -f dev-server.log ]; then
                        mv dev-server.log dev-server-${BUILD_NUMBER}.log
                    fi
                '''
                
                // Archive the logs as artifacts
                archiveArtifacts artifacts: '**/dev-server-*.log', allowEmptyArchive: true
            }
            
            cleanWs()
        }
        failure {
            script {
                echo """
                Build failed! Common troubleshooting steps:
                1. Check Node.js installation: node --version
                2. Check npm installation: npm --version
                3. Review archived logs for errors
                4. Verify package.json exists and is valid
                5. Check npm scripts in package.json
                6. Ensure all required environment variables are set
                """
            }
        }
    }
}