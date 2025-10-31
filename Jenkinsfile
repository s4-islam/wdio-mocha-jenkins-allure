pipeline {
    agent any

    tools {
        nodejs NodeJS
    }

    environment {
        PROJECT_NAME = "WDIO-Mocha-Jenkins-Allure"
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
            echo 'Cleaning workspace and publishing allure report...'
            archiveArtifacts artifacts: 'allure-results/**', fingerprint: true
            allure([
                reportBuildPolicy: 'ALWAYS',
                results: [[path: "${ALLURE_RESULTS_DIR}"]]
            ])
        }

        success {
            echo "Pipeline for ${PROJECT_NAME} completed successfully!"
        }

        failure {
            echo "Pipeline for ${PROJECT_NAME} failed! Check console and Allure report."
        }
    }
}
