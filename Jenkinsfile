pipeline {
    agent none

    tools {
        nodejs NodeJS
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
                echo 'Generating Allure Report...'
                sh 'npx allure generate ${ALLURE_RESULTS_DIR} --clean -o ${ALLURE_REPORT_DIR}'
            }
        }
    }

    post {

        always {
            node('any') {
                cleanWs()
                allure([
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: 'allure-report']]
                ])
            }
            echo 'Cleaning workspace and publishing allure report...'
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
        }

        success {
            echo "Pipeline for WDIO-Mocha-Jenkins-Allure completed successfully!"
        }

        failure {
            echo "Pipeline for WDIO-Mocha-Jenkins-Allure failed! Check console and Allure report."
        }
    }
}
