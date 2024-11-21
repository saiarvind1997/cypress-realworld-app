pipeline {
    agent any
    
    environment {
        NODE_HOME = "${WORKSPACE}/node-v18.17.0-linux-x64"
        YARN_HOME = "${WORKSPACE}/yarn"
        PATH = "${NODE_HOME}/bin:${YARN_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        # Download and install Node.js
                        curl -fsSL https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.gz | tar xz
                        
                        # Download and install Yarn
                        curl -fsSL https://github.com/yarnpkg/yarn/releases/download/v1.22.19/yarn-v1.22.19.tar.gz | tar xz
                        mv yarn-v1.22.19 yarn
                        
                        # Verify installations
                        echo "Node version:"
                        ${NODE_HOME}/bin/node --version
                        
                        echo "Yarn version:"
                        ${YARN_HOME}/bin/yarn --version
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        echo "Installing dependencies..."
                        yarn install
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        # Seed the database
                        yarn db:seed:dev
                        
                        # Start app in CI mode and run tests
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
            sh '''
                echo "Cleaning up processes..."
                pkill -f "start:ci" || true
            '''
            cleanWs()
        }
    }
}