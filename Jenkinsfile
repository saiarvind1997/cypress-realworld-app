pipeline {
    agent any
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        # Download and setup Node.js 18
                        curl -fsSL https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.gz | tar xz
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        
                        # Download and setup Yarn
                        curl -fsSL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor > /usr/share/keyrings/yarn-archive-keyring.gpg || true
                        curl -o yarn.tar.gz https://github.com/yarnpkg/yarn/releases/download/v1.22.19/yarn-v1.22.19.tar.gz
                        tar zvxf yarn.tar.gz
                        export PATH="$PWD/yarn-v1.22.19/bin:$PATH"
                        
                        # Verify installations
                        echo "Node version:"
                        node --version
                        echo "NPM version:"
                        npm --version
                        echo "Yarn version:"
                        yarn --version
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        # Set PATH for Node and Yarn
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        export PATH="$PWD/yarn-v1.22.19/bin:$PATH"
                        
                        # Install dependencies with Yarn
                        yarn install
                        
                        # Seed the database
                        yarn db:seed:dev
                    '''
                }
            }
        }

        stage('Start App and Test') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        export PATH="$PWD/yarn-v1.22.19/bin:$PATH"
                        
                        # Start the app in the background
                        yarn start:ci &
                        sleep 30
                        
                        # Run tests
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
                    # Cleanup processes
                    pkill -f "yarn start:ci" || true
                '''
                cleanWs()
            }
        }
    }
}