pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        NODE_PATH = "${WORKSPACE}/.nodejs" // Local to workspace
        PATH = "${WORKSPACE}/.nodejs/bin:${env.PATH}"
    }
    
    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    sh '''
                        # Create local directory for Node.js
                        mkdir -p ${NODE_PATH}
                        
                        # Download Node.js using curl
                        echo "Downloading Node.js..."
                        curl -L -o node.tar.xz https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz
                        
                        # Extract Node.js to local directory
                        echo "Extracting Node.js..."
                        tar -xJf node.tar.xz -C ${NODE_PATH} --strip-components=1
                        rm node.tar.xz
                        
                        # Add local Node.js to PATH and verify
                        export PATH="${NODE_PATH}/bin:$PATH"
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
                        
                        # Clean and install dependencies
                        echo "Installing dependencies..."
                        rm -rf node_modules package-lock.json
                        npm install --no-audit --no-fund
                    '''
                }
            }
        }
        
        stage('Start Development Server') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        # Kill any existing process
                        lsof -ti:3000 | xargs kill -9 || true
                        
                        # Start the dev server
                        echo "Starting development server..."
                        nohup npm run dev > dev-server.log 2>&1 &
                        echo $! > .dev-server.pid
                        
                        # Wait for server to start
                        echo "Waiting for server to start..."
                        sleep 30
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
                    # Cleanup processes
                    if [ -f .dev-server.pid ]; then
                        kill -9 $(cat .dev-server.pid) || true
                        rm .dev-server.pid
                    fi
                '''
                cleanWs()
            }
        }
    }
}