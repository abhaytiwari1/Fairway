pipeline {
    agent any

    environment {
        SF_USERNAME = 'abhay.b.tiwari.631b9ea19687@agentforce.com'
        SF_CONSUMER_KEY = credentials('consumer_key')   // Your connected app consumer key
        SF_CLI = '"C:\\Program Files\\sf\\bin\\sf.cmd"' // Full path to Salesforce CLI
    }

    stages {

        stage('Authenticate Salesforce') {
            steps {
                // Use Jenkins secret file for JWT key
                withCredentials([file(credentialsId: 'SF_JWT_KEY', variable: 'JWT_KEY_FILE')]) {
                    bat """
                    %SF_CLI% force:auth:jwt:grant ^
                    --client-id %SF_CONSUMER_KEY% ^
                    --jwt-key-file %JWT_KEY_FILE% ^
                    --username %SF_USERNAME% ^
                    --instance-url https://login.salesforce.com ^
                    --set-default-dev-hub ^
                    --alias projectdemosfdc
                    """
                }
            }
        }

        stage('Deploy to Org') {
            steps {
                bat """
                %SF_CLI% project deploy start ^
                --source-dir force-app ^
                --target-org projectdemosfdc ^
                --wait 10
                """
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                %SF_CLI% apex run test ^
                --target-org projectdemosfdc ^
                --wait 10 ^
                --result-format human
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Deployment succeeded ✅"
        }
        failure {
            echo "Deployment failed ❌"
        }
    }
}