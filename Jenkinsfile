pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        ALLURE_RESULTS_DIR = "allure-results"
        ALLURE_REPORT_DIR = "allure-report"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out GitHub repo'
                checkout scm
            }
        }

        stage('Fix Dependencies') {
            steps {
                echo 'Installing system dependencies for Node.js'
                sh '''
                    # Check if we can install system packages (if running as root or with sudo)
                    if command -v apt-get >/dev/null 2>&1; then
                        apt-get update && apt-get install -y libatomic1 || echo "Cannot install system packages, trying alternative approach"
                    elif command -v yum >/dev/null 2>&1; then
                        yum install -y libatomic || echo "Cannot install system packages, trying alternative approach" 
                    fi
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing necessary dependencies for project'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests from WDIO+Mocha project'
                sh 'npm test'
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
