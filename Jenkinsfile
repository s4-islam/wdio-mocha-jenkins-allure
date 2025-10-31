pipeline {
    agent any

    environment {
        ALLURE_RESULTS_DIR = "allure-results"
        ALLURE_REPORT_DIR = "allure-report"
        NODE_VERSION = "20.18.0"
        PATH = "${env.WORKSPACE}/node-v${NODE_VERSION}-linux-x64/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out GitHub repo'
                checkout scm
            }
        }

        stage('Install Node.js and Dependencies') {
            steps {
                echo "Installing Node.js ${NODE_VERSION} and project dependencies"
                sh '''
                    # Download and install Node.js 20.x (using .tar.gz instead of .tar.xz)
                    echo "Downloading Node.js ${NODE_VERSION}..."
                    curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz -o node.tar.gz
                    
                    echo "Extracting Node.js..."
                    tar -xzf node.tar.gz
                    
                    echo "Verifying Node.js installation..."
                    ./node-v${NODE_VERSION}-linux-x64/bin/node --version
                    ./node-v${NODE_VERSION}-linux-x64/bin/npm --version
                    
                    echo "Installing project dependencies..."
                    ./node-v${NODE_VERSION}-linux-x64/bin/npm install
                    
                    echo "Node.js and dependencies installed successfully!"
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests from WDIO+Mocha project'
                sh 'node --version && npm --version && npm test'
            }
        }

        stage('Generate Allure Report') {
            steps {
                echo 'Generating Allure Report using npm script...'
                sh 'npm run allure-generate || echo "Allure generation failed, but continuing pipeline"'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Archiving test results and reports...'
            // Archive allure results if they exist
            script {
                if (fileExists('allure-results')) {
                    archiveArtifacts artifacts: 'allure-results/**', fingerprint: true, allowEmptyArchive: true
                    echo 'Allure results archived'
                } else {
                    echo 'No allure-results directory found'
                }
                
                if (fileExists('allure-report')) {
                    archiveArtifacts artifacts: 'allure-report/**', fingerprint: true, allowEmptyArchive: true
                    echo 'Allure report archived'
                } else {
                    echo 'No allure-report directory found'
                }
            }
            
            // Only try to publish Allure if the plugin is properly configured
            script {
                try {
                    allure([
                        reportBuildPolicy: 'ALWAYS',
                        results: [[path: env.ALLURE_RESULTS_DIR]]
                    ])
                    echo 'Allure report published successfully'
                } catch (Exception e) {
                    echo "Allure plugin not configured properly: ${e.getMessage()}"
                    echo 'Allure results have been archived as artifacts instead'
                }
            }
            
            echo "Pipeline for WDIO-Mocha-Jenkins-Allure completed successfully!"
            cleanWs()
        }

        failure {
            echo 'Pipeline failed! Still archiving available test results...'
            // Archive whatever results we have, even on failure
            script {
                if (fileExists('allure-results')) {
                    archiveArtifacts artifacts: 'allure-results/**', fingerprint: true, allowEmptyArchive: true
                    echo 'Allure results archived (despite pipeline failure)'
                }
                
                if (fileExists('allure-report')) {
                    archiveArtifacts artifacts: 'allure-report/**', fingerprint: true, allowEmptyArchive: true
                    echo 'Allure report archived (despite pipeline failure)'
                }
            }
            
            echo "Pipeline for WDIO-Mocha-Jenkins-Allure failed! Check console and archived artifacts."
            cleanWs()
        }
    }
}
