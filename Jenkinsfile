pipeline {
    agent any

    environment {
        SF_USERNAME     = credentials('user_name')
        SF_CONSUMER_KEY = credentials('consumer_key')
        SF_CLI          = 'C:/Program Files/sf/bin/sf.cmd'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/abhaytiwari1/Fairway.git'
            }
        }

        stage('Authenticate Salesforce') {
            steps {
                withCredentials([file(credentialsId: 'jwt_key', variable: 'JWT_KEY_FILE')]) {
                    bat """
                    echo Authenticating to Salesforce...

                    "%SF_CLI%" org login jwt ^
                    --client-id %SF_CONSUMER_KEY% ^
                    --jwt-key-file "%JWT_KEY_FILE%" ^
                    --username %SF_USERNAME% ^
                    --instance-url https://login.salesforce.com ^
                    --alias projectdemosfdc

                    echo Authentication Successful
                    """
                }
            }
        }

        stage('Validate Deployment (Dry Run - Non Blocking)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat """
                    echo Running Validation (Dry Run)...

                    "%SF_CLI%" project deploy start ^
                    --target-org projectdemosfdc ^
                    --dry-run ^
                    --test-level RunLocalTests ^
                    --wait 20

                    echo Validation Completed
                    """
                }
            }
        }

        stage('Deploy to Org (With Tests)') {
            steps {
                bat """
                echo Deploying Metadata with Tests...

                "%SF_CLI%" project deploy start ^
                --target-org projectdemosfdc ^
                --test-level RunLocalTests ^
                --wait 30

                echo Deployment Completed
                """
            }
        }

        stage('Run Apex Tests (JUnit Report)') {
            steps {
                bat """
                echo Running Apex Tests for Reporting...

                "%SF_CLI%" apex run test ^
                --target-org projectdemosfdc ^
                --test-level RunLocalTests ^
                --result-format junit ^
                --output-dir test-results ^
                --wait 30

                echo Apex Tests Completed
                """
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'test-results\\*.xml'
        }
        success {
            echo '✅ Salesforce deployment and tests completed successfully'
        }
        failure {
            echo '❌ Pipeline failed. Likely due to test coverage < 75% or deployment errors.'
        }
    }
}