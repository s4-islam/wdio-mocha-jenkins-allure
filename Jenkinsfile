pipeline {
    agent any

    environment {
        PROJECT_NAME = "WDIO-Mocha-Jenkins-Allure"
        TEST_COMMAND = "npm test"
        INSTALL_COMMAND = "npm install"
        ALLURE_RESULTS_DIR = "allure-results"
        ALLURE_REPORT_DIR = "allure-report"
        ALLURE_GENERATE_CMD = "npx allure generate ${ALLURE_RESULTS_DIR} --clean -o ${ALLURE_REPORT_DIR}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out GitHub repo"
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing necessary dependencies for project"
                sh "${INSTALL_COMMAND}"
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running tests from WDIO+Mocha project"
                sh "${TEST_COMMAND}"
            }
        }

        stage('Generate Allure Report') {
            steps {
                echo "📊 Generating Allure Report..."
                sh "${ALLURE_GENERATE_CMD}"
            }
        }
    }

    post {

        always {
            echo "Cleaning workspace and archiving results..."
            archiveArtifacts artifacts: "${ALLURE_RESULTS_DIR}/**", fingerprint: true

            echo "Publishing Allure Report from ${ALLURE_RESULTS_DIR}"
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
