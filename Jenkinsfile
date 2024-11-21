pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18.17.0'
        YARN_VERSION = '1.22.19'
        NODE_PATH = "${WORKSPACE}/node-v${NODE_VERSION}-linux-x64/bin"
        YARN_PATH = "${WORKSPACE}/yarn-v${YARN_VERSION}/bin"
        PATH = "${NODE_PATH}:${YARN_PATH}:${env.PATH}"
        DISPLAY = ':99'
    }
    
    stages {
        stage('Setup') {
            steps {
                script {
                    sh '''
                        set -e
                        
                        # Clean previous installations
                        rm -rf node-* yarn-* yarn.tar.gz xvfb

                        # Download Xvfb standalone binary
                        mkdir -p bin
                        curl -L -o bin/Xvfb https://github.com/electron/electron/raw/main/shell/browser/resources/xvfb/Xvfb
                        chmod +x bin/Xvfb
                        export PATH="${WORKSPACE}/bin:${PATH}"
                        
                        # Install Node.js
                        curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz | tar xz
                        
                        # Install Yarn
                        curl -fsSL https://github.com/yarnpkg/yarn/releases/download/v${YARN_VERSION}/yarn-v${YARN_VERSION}.tar.gz | tar xz
                        
                        # Verify installations
                        if ! ${NODE_PATH}/node --version || ! ${YARN_PATH}/yarn --version; then
                            echo "Failed to install Node.js or Yarn"
                            exit 1
                        fi
                        
                        # Start Xvfb
                        Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
                        
                        # Wait for Xvfb to start
                        sleep 3
                    '''
                }
            }
        }
    }
    # Rest of pipeline remains the same...
}