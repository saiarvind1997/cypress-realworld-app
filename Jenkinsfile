pipeline {
    agent any

    environment {
        CYPRESS_CRASH_REPORTS = '0'
        NODE_OPTIONS = '--max-old-space-size=4096'
        DEBUG = 'cypress:*'  // Enable Cypress debugging
    }

    stages {
        stage('Install') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                sh 'yarn install --frozen-lockfile'
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        sh 'yarn test:unit:ci'
                    } catch (Exception e) {
                        unstable('Unit tests failed')
                    }
                }
            }
        }

        stage('Component Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        sh 'yarn test:component:ci'
                    } catch (Exception e) {
                        unstable('Component tests failed')
                    }
                }
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'cypress/included:13.16.0'
                    args '--entrypoint='
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        sh '''
                            # Print environment for debugging
                            echo "Node version:"
                            node --version
                            echo "NPM version:"
                            npm --version
                            echo "Yarn version:"
                            yarn --version
                            
                            # Clean start
                            pkill -f "node" || true
                            
                            # Setup test environment
                            echo "Setting up test environment..."
                            yarn db:seed:dev
                            yarn predev:cognito:ci
                            
                            # Start the app in debug mode
                            echo "Starting application..."
                            DEBUG=* yarn start:ci &
                            
                            # Wait and verify services
                            echo "Waiting for services..."
                            sleep 30
                            
                            # Check if services are running
                            echo "Checking services..."
                            netstat -tulpn | grep LISTEN
                            
                            # Run tests with detailed logging
                            echo "Running Cypress tests..."
                            yarn cypress run --browser chrome --config video=true --reporter-options "reportDir=cypress/reports" || {
                                echo "Cypress tests failed. Checking logs..."
                                cat cypress/reports/*.log || true
                                cat ~/.npm/_logs/*-debug.log || true
                                exit 1
                            }
                        '''
                    } catch (Exception e) {
                        echo "Test execution failed with error: ${e.getMessage()}"
                        unstable('E2E tests failed')
                    } finally {
                        sh '''
                            echo "Collecting logs..."
                            mkdir -p test-output
                            cp -r cypress/reports/* test-output/ || true
                            cp -r cypress/videos/* test-output/ || true
                            cp -r cypress/screenshots/* test-output/ || true
                            
                            echo "Cleaning up processes..."
                            pkill -f "node" || true
                            sleep 2
                            pkill -9 -f "node" || true
                        '''
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '''
                        test-output/**/*,
                        cypress/videos/**/*.mp4,
                        cypress/screenshots/**/*.png,
                        cypress/logs/**/*
                    ''', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '''
                cypress/videos/**/*.mp4,
                cypress/screenshots/**/*.png,
                test-output/**/*
            ''', allowEmptyArchive: true
        }
        unstable {
            echo 'Tests failed - check the artifacts for detailed logs'
        }
    }
}

// pipeline {
//     agent any

//     environment {
//         CYPRESS_CRASH_REPORTS = '0'
//         NODE_OPTIONS = '--max-old-space-size=4096'
//     }

//     stages {
//         stage('Install') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh 'yarn install --frozen-lockfile'
//             }
//         }

//         stage('Unit Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh 'yarn test:unit:ci'
//                     } catch (Exception e) {
//                         unstable('Unit tests failed')
//                     }
//                 }
//             }
//         }

//         stage('Component Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh 'yarn test:component:ci'
//                     } catch (Exception e) {
//                         unstable('Component tests failed')
//                     }
//                 }
//             }
//         }

//         stage('E2E Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh '''
//                             # Ensure clean start
//                             pkill -f "node" || true
                            
//                             # Setup database and AWS mock
//                             echo "Setting up test environment..."
//                             yarn db:seed:dev
//                             yarn predev:cognito:ci
                            
//                             # Start backend first
//                             echo "Starting backend..."
//                             yarn start:api &
                            
//                             # Wait for backend
//                             echo "Waiting for backend..."
//                             for i in {1..30}; do
//                                 if nc -z localhost 3001; then
//                                     echo "Backend is ready!"
//                                     break
//                                 fi
//                                 if [ $i -eq 30 ]; then
//                                     echo "Backend failed to start"
//                                     exit 1
//                                 fi
//                                 sleep 2
//                             done
                            
//                             # Start frontend
//                             echo "Starting frontend..."
//                             yarn start:react &
                            
//                             # Wait for frontend
//                             echo "Waiting for frontend..."
//                             for i in {1..30}; do
//                                 if nc -z localhost 3000; then
//                                     echo "Frontend is ready!"
//                                     break
//                                 fi
//                                 if [ $i -eq 30 ]; then
//                                     echo "Frontend failed to start"
//                                     exit 1
//                                 fi
//                                 sleep 2
//                             done
                            
//                             echo "Both services are up, starting tests..."
//                             yarn cypress run
//                         '''
//                     } catch (Exception e) {
//                         unstable('E2E tests failed')
//                     } finally {
//                         sh '''
//                             echo "Cleaning up processes..."
//                             pkill -f "node" || true
//                             sleep 2
//                             pkill -9 -f "node" || true
//                         '''
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             archiveArtifacts artifacts: '''
//                 cypress/videos/**/*.mp4,
//                 cypress/screenshots/**/*.png
//             ''', allowEmptyArchive: true
//         }
//     }
// }

// pipeline {
//     agent any

//     environment {
//         CYPRESS_CRASH_REPORTS = '0'
//         NODE_OPTIONS = '--max-old-space-size=4096'
//     }

//     stages {
//         stage('Install') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh 'yarn install --frozen-lockfile'
//             }
//         }

//         stage('Unit Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh '''
//                             echo "Running Unit Tests..."
//                             yarn test:unit:ci
//                         '''
//                     } catch (Exception e) {
//                         unstable('Unit tests failed')
//                     }
//                 }
//             }
//         }

//         stage('Component Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh '''
//                             echo "Running Component Tests..."
//                             yarn test:component:ci
//                         '''
//                     } catch (Exception e) {
//                         unstable('Component tests failed')
//                     }
//                 }
//             }
//         }

//         stage('E2E Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 script {
//                     try {
//                         sh '''
//                             echo "Starting application in CI mode..."
//                             yarn prestart:ci
//                             yarn start:ci &
                            
//                             echo "Waiting for application to start..."
//                             sleep 45
                            
//                             echo "Running E2E Tests..."
//                             yarn test:headless
//                         '''
//                     } catch (Exception e) {
//                         unstable('E2E tests failed')
//                     } finally {
//                         sh 'pkill -f "node" || true'
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             archiveArtifacts artifacts: '''
//                 cypress/videos/**/*.mp4,
//                 cypress/screenshots/**/*.png
//             ''', allowEmptyArchive: true
//         }
//         unstable {
//             echo 'One or more test stages failed'
//         }
//         success {
//             echo 'All test stages passed successfully'
//         }
//         failure {
//             echo 'Pipeline failed'
//         }
//     }
// }


// ???///?////////////////////////////////////////?

// pipeline {
//     agent any

//     stages {
//         stage('Install') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh 'yarn install --frozen-lockfile'
//             }
//         }

//         stage('Unit Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh 'yarn test:unit:ci'  // Using CI version of unit tests
//             }
//         }

//         stage('Component Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh 'yarn test:component:ci'  // Using CI version of component tests
//             }
//         }

//         stage('E2E Tests') {
//             agent {
//                 docker {
//                     image 'cypress/included:13.16.0'
//                     args '--entrypoint='
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh '''
//                     # Setup and start the application in CI mode
//                     yarn prestart:ci
//                     yarn start:ci &
                    
//                     # Wait for services
//                     sleep 45
                    
//                     # Run E2E tests headless
//                     yarn test:headless
//                 '''
//             }
//             post {
//                 always {
//                     sh 'pkill -f "node" || true'
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             archiveArtifacts artifacts: 'cypress/videos/**/*.mp4,cypress/screenshots/**/*.png', allowEmptyArchive: true
//         }
//     }
// }