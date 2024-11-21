pipeline {
    agent any
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        curl -fsSL https://nodejs.org/dist/v16.18.0/node-v16.18.0-linux-x64.tar.gz | tar xz
                        export PATH=$PWD/node-v16.18.0-linux-x64/bin:$PATH
                        node --version
                        npm --version
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v16.18.0-linux-x64/bin:$PATH
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