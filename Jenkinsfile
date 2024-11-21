pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        # Download and install Node.js
                        curl -fsSL https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.gz | tar xz
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        
                        # Install Yarn
                        npm install -g yarn
                        
                        # Show versions
                        node --version
                        yarn --version
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        yarn install
                    '''
                }
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        
                        # Seed the database with development data
                        yarn db:seed:dev
                        
                        # Start the app in CI mode and run tests
                        yarn start:ci &
                        sleep 30
                        yarn test:headless
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh 'pkill -f "start:ci" || true'
            cleanWs()
        }
    }
}