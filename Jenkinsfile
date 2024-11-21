pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        NODE_PATH = "${WORKSPACE}/.nodejs"
        PATH = "${WORKSPACE}/.nodejs/bin:${env.PATH}"
    }
    
    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    sh '''
                        # Create local directory for Node.js
                        mkdir -p ${NODE_PATH}
                        
                        # Download Node.js using .tar.gz instead of .tar.xz
                        echo "Downloading Node.js..."
                        curl -L -o node.tar.gz https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz
                        
                        # Extract Node.js to local directory
                        echo "Extracting Node.js..."
                        tar -xzf node.tar.gz -C ${NODE_PATH} --strip-components=1
                        rm node.tar.gz
                        
                        # Add local Node.js to PATH and verify
                        export PATH="${NODE_PATH}/bin:$PATH"
                        node --version || {
                            echo "Node.js installation failed"
                            exit 1
                        }
                        
                        npm --version || {
                            echo "npm installation failed"
                            exit 1
                        }
                        
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
                        
                        echo "Installing dependencies..."
                        rm -rf node_modules package-lock.json
                        
                        # Install with more memory allocation
                        export NODE_OPTIONS="--max_old_space_size=4096"
                        npm install --no-audit --no-fund
                        
                        # Verify installation
                        echo "Installed packages:"
                        npm list --depth=0
                    '''
                }
            }
        }
        
        stage('Start Development Server') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        # Try to free up port 3000 if in use
                        lsof -ti:3000 | xargs kill -9 || true
                        
                        echo "Starting development server..."
                        nohup npm run dev > dev-server.log 2>&1 &
                        echo $! > .dev-server.pid
                        
                        # Wait and check server status
                        echo "Waiting for server to start..."
                        sleep 30
                        
                        if ! cat dev-server.log; then
                            echo "Could not read server logs"
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
                        export NODE_OPTIONS="--max_old_space_size=4096"
                        
                        echo "Running tests..."
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
                    # Cleanup processes
                    if [ -f .dev-server.pid ]; then
                        pid=$(cat .dev-server.pid)
                        kill -9 $pid || true
                        rm .dev-server.pid
                    fi
                    
                    # Archive logs if they exist
                    if [ -f dev-server.log ]; then
                        echo "Server Logs:"
                        cat dev-server.log
                    fi
                '''
                cleanWs()
            }
        }
        failure {
            script {
                echo """
                Build failed! Debug information:
                - Check if Node.js was installed correctly: ls -la ${NODE_PATH}/bin
                - Check npm installation: ${NODE_PATH}/bin/npm --version
                - Check server logs above for any errors
                - Verify port 3000 is available: lsof -i:3000
                """
            }
        }
    }
}