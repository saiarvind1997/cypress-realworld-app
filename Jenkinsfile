pipeline {
    agent any
    
    tools {
        nodejs 'Node'  // Make sure to configure Node.js in Jenkins Global Tool Configuration
    }
    
    environment {
        // Customize these environment variables as needed
        NODE_ENV = 'development'
        CI = 'true'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Run Development Server') {
            steps {
                script {
                    // Start the dev server in the background
                    sh '''
                        nohup npm run dev &
                        echo $! > .dev-server.pid
                        # Give the server some time to start
                        sleep 30
                    '''
                }
            }
        }
        
        stage('Run Headless Tests') {
            steps {
                sh 'npm run test:headless'
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup: Stop the dev server if it's running
                sh '''
                    if [ -f .dev-server.pid ]; then
                        pid=$(cat .dev-server.pid)
                        kill $pid || true
                        rm .dev-server.pid
                    fi
                '''
            }
            
            // Clean workspace
            cleanWs()
        }
    }
}