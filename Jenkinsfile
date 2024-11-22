pipeline {
    agent any

    environment {
        CYPRESS_CRASH_REPORTS = '0'
        NODE_OPTIONS = '--max-old-space-size=4096'
    }

    stages {
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    yarn install --frozen-lockfile
                    yarn add -D @babel/plugin-proposal-private-property-in-object
                    # Install node-fetch for health checks
                    yarn add --dev node-fetch
                '''
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    mkdir -p test-results/unit
                    yarn test:unit
                '''
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '-u root'
                }
            }
            steps {
                script {
                    try {
                        // Create a health check script
                        writeFile file: 'healthCheck.js', text: '''
                            const fetch = require('node-fetch');
                            
                            async function checkHealth(url, maxAttempts = 30) {
                                for (let i = 1; i <= maxAttempts; i++) {
                                    try {
                                        await fetch(url);
                                        console.log(`${url} is up!`);
                                        return true;
                                    } catch (error) {
                                        console.log(`Attempt ${i}: Waiting for ${url}...`);
                                        if (i === maxAttempts) {
                                            console.log(`Failed to connect to ${url}`);
                                            process.exit(1);
                                        }
                                        await new Promise(res => setTimeout(res, 1000));
                                    }
                                }
                                return false;
                            }
                            
                            (async () => {
                                await checkHealth('http://localhost:3001');  // Check backend
                                await checkHealth('http://localhost:3000');  // Check frontend
                            })();
                        '''

                        // Start the application and run health checks
                        sh '''
                            echo "Starting application..."
                            yarn start &
                            
                            echo "Checking application health..."
                            node healthCheck.js
                            
                            echo "Running Cypress tests..."
                            mkdir -p test-results/cypress
                            yarn cypress run \
                                --config video=true \
                                --reporter junit \
                                --reporter-options "mochaFile=test-results/cypress/results-[hash].xml"
                        '''
                    } finally {
                        sh '''
                            echo "Cleaning up processes..."
                            pkill -f "yarn start" || true
                            pkill -f "node" || true
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/test-results/**/*.xml'
            
            archiveArtifacts artifacts: '''
                cypress/videos/**/*.mp4,
                cypress/screenshots/**/*.png,
                test-results/**/*.xml
            ''', allowEmptyArchive: true
            
            sh '''
                mkdir -p logs
                if [ -f yarn-error.log ]; then
                    cp yarn-error.log logs/
                fi
                if [ -d cypress/logs ]; then
                    cp cypress/logs/* logs/ || true
                fi
            '''
            archiveArtifacts artifacts: 'logs/**/*', allowEmptyArchive: true
        }
    }
}