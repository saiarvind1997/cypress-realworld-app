pipeline {
    agent any
    
    stages {
        stage('Install Node.js') {
            steps {
                sh '''
                    # Install Node.js using package manager
                    sudo apt-get update
                    sudo apt-get install -y nodejs npm
                    
                    # Verify installation
                    node --version
                    npm --version
                '''
            }
        }
        
        stage('Build and Test') {
            steps {
                sh '''
                    npm install
                    npm run dev &
                    sleep 30
                    npm run test:headless
                '''
            }
        }
    }
    
    post {
        always {
            // Kill any remaining npm processes
            sh 'pkill -f "npm run dev" || true'
            cleanWs()
        }
    }
}