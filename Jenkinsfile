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
                        mkdir -p ${NODE_PATH}
                        
                        echo "Downloading Node.js..."
                        curl -L -o node.tar.gz https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz
                        
                        echo "Extracting Node.js..."
                        tar -xzf node.tar.gz -C ${NODE_PATH} --strip-components=1
                        rm node.tar.gz
                        
                        export PATH="${NODE_PATH}/bin:$PATH"
                        node --version
                        npm --version
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
                        npm install
                        
                        # Verify Cypress installation
                        echo "Verifying Cypress installation..."
                        npx cypress verify
                    '''
                }
            }
        }
        
        stage('Build App') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        npm run build
                    '''
                }
            }
        }
        
        stage('Start Backend') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        echo "Starting backend server..."
                        npm run start:api &
                        echo $! > .backend-pid
                        
                        # Wait for backend to be ready
                        echo "Waiting for backend to start..."
                        sleep 30
                    '''
                }
            }
        }
        
        stage('Run Cypress Tests') {
            steps {
                script {
                    sh '''
                        export PATH="${NODE_PATH}/bin:$PATH"
                        
                        echo "Running Cypress tests..."
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
                    # Kill backend process if running
                    if [ -f .backend-pid ]; then
                        kill -9 $(cat .backend-pid) || true
                        rm .backend-pid
                    fi
                '''
                cleanWs()
            }
        }
    }
}