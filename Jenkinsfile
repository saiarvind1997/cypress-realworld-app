pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        // Set up custom paths for local installation
        NODE_PATH = "${WORKSPACE}/nodejs"
        PATH = "${WORKSPACE}/nodejs/bin:${env.PATH}"
    }
    
    stages {
        stage('Install Node.js') {
            steps {
                script {
                    sh '''
                        # Create directory for Node.js
                        mkdir -p ${NODE_PATH}
                        
                        # Download and extract Node.js binary
                        curl -o node.tar.xz https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.xz
                        tar -xJf node.tar.xz -C ${NODE_PATH} --strip-components=1
                        rm node.tar.xz
                        
                        # Add node and npm to PATH
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        # Verify installation
                        node --version
                        npm --version
                        
                        # Configure npm to use local prefix
                        npm config set prefix "${NODE_PATH}"
                    '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        # Clear existing modules if any
                        rm -rf node_modules package-lock.json
                        
                        # Install dependencies
                        npm install
                        
                        # Verify installation
                        npm list --depth=0
                    '''
                }
            }
        }
        
        stage('Run Development Server') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        # Check if port 3000 is in use
                        if netstat -tln | grep ':3000 '; then
                            # Find and kill process using port 3000
                            processId=$(lsof -t -i:3000) || true
                            if [ ! -z "$processId" ]; then
                                kill -9 $processId
                            fi
                        fi
                        
                        # Start the dev server
                        nohup npm run dev > dev-server.log 2>&1 &
                        echo $! > .dev-server.pid
                        
                        # Wait for server to start
                        count=0
                        while [ $count -lt 30 ]; do
                            if grep -q "ready" dev-server.log 2>/dev/null; then
                                echo "Server started successfully"
                                break
                            fi
                            count=$((count + 1))
                            echo "Waiting for server to start... ($count/30)"
                            sleep 2
                        done
                        
                        # Check if server started
                        if [ $count -eq 30 ]; then
                            echo "Server failed to start. Log output:"
                            cat dev-server.log
                            exit 1
                        fi
                    '''
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        npm run test:headless
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh '''
                    # Cleanup: Stop the dev server
                    if [ -f .dev-server.pid ]; then
                        pid=$(cat .dev-server.pid)
                        kill -9 $pid || true
                        rm .dev-server.pid
                    fi
                    
                    # Archive logs
                    if [ -f dev-server.log ]; then
                        mv dev-server.log dev-server-${BUILD_NUMBER}.log
                    fi
                '''
                
                // Archive logs as artifacts
                archiveArtifacts artifacts: '**/dev-server-*.log', allowEmptyArchive: true
            }
            
            // Clean workspace
            cleanWs()
        }
        
        failure {
            echo '''
                Build failed! Common issues to check:
                1. Node.js installation - check dev-server log
                2. Dependencies installation
                3. Port 3000 availability
                4. Test execution
                
                See archived logs for more details.
            '''
        }
    }
}