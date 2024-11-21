pipeline {
    agent any
    
    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        # Install system dependencies without sudo
                        apt-get update
                        apt-get install -y \
                            libgtk2.0-0 \
                            libgtk-3-0 \
                            libgbm-dev \
                            libnotify-dev \
                            libgconf-2-4 \
                            libnss3 \
                            libxss1 \
                            libasound2 \
                            libxtst6 \
                            xauth \
                            xvfb \
                            curl
                            
                        # Install Node.js
                        curl -fsSL https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.gz | tar xz
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        
                        # Install Yarn
                        npm install -g yarn
                        
                        # Verify installations
                        echo "Node version:"
                        node --version
                        echo "Yarn version:"
                        yarn --version
                    '''
                }
            }
        }

        stage('Install Project Dependencies') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        yarn install
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        export PATH=$PWD/node-v18.17.0-linux-x64/bin:$PATH
                        
                        # Seed the database
                        yarn db:seed:dev
                        
                        # Start app and run tests with xvfb
                        xvfb-run --server-args="-screen 0 1280x720x24" yarn start:ci &
                        sleep 30
                        xvfb-run --server-args="-screen 0 1280x720x24" yarn test:headless
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