pipeline {
    agent any
    
    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    sh '''
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        export NVM_DIR="$HOME/.nvm"
                        . "$NVM_DIR/nvm.sh"
                        nvm install 16
                        node --version
                        npm --version
                    '''
                }
            }
        }

        stage('Install & Test') {
            steps {
                script {
                    sh '''
                        export NVM_DIR="$HOME/.nvm"
                        . "$NVM_DIR/nvm.sh"
                        npm install
                        npm run cypress:run
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}