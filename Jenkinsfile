pipeline {
    agent any
    
    environment {
        NODE_VERSION = '16.18.0'  // Using the version recommended by Cypress RWA
        NODE_PATH = "${WORKSPACE}/.nodejs"
        PATH = "${WORKSPACE}/.nodejs/bin:${env.PATH}"
    }
    
    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    sh '''
                        rm -rf ${NODE_PATH}
                        mkdir -p ${NODE_PATH}
                        
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        export NVM_DIR="$HOME/.nvm"
                        [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"
                        
                        nvm install 16
                        nvm use 16
                        
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
                        npm install
                        
                        echo "Verifying Cypress binary installation..."
                        ./node_modules/.bin/cypress verify
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        echo "Starting backend..."
                        npm run start:api:ci &
                        sleep 10
                        
                        echo "Running Cypress tests..."
                        npm run cypress:run
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
                    pkill -f "npm run start:api:ci" || true
                '''
                cleanWs()
            }
        }
    }
}