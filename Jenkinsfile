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
                    "%SF_CLI%" org login jwt ^
                    --client-id %SF_CONSUMER_KEY% ^
                    --jwt-key-file "%JWT_KEY_FILE%" ^
                    --username %SF_USERNAME% ^
                    --instance-url https://login.salesforce.com ^
                    --alias projectdemosfdc
                    """
                }
            }
        }

        stage('Deploy to Org') {
            steps {
                bat """
                echo Deploying Metadata...

                "%SF_CLI%" deploy metadata ^
                --target-org projectdemosfdc ^
                --wait 10

                echo Deployment Completed
                """
            }
        }

        stage('Run Apex Tests') {
            steps {
                bat """
                echo Running Apex Tests...

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
            echo '❌ Pipeline failed. Check logs above for details.'
        }
    }
}