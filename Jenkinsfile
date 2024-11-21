pipeline {
    agent any
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        curl -fsSL https://nodejs.org/dist/v16.18.0/node-v16.18.0-linux-x64.tar.gz | tar xz
                        export PATH=$PWD/node-v16.18.0-linux-x64/bin:$PATH

                         curl -o- -L https://yarnpkg.com/install.sh | bash
                        export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
                        
                        # Verify installations
                        node --version
                        npm --version
                        yarn --version

                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v16.18.0-linux-x64/bin:$PATH
                        npm install --legacy-peer-deps
                        nohup npm run dev &
                        sleep 10
                        npm run test:headless
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