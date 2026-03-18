pipeline {
    agent any

    environment {
        SF_USERNAME = 'abhay.b.tiwari.631b9ea19687@agentforce.com'
        SF_CONSUMER_KEY = credentials('consumer_key')
    }

    stages {


        stage('Authenticate Salesforce') {
            steps {
                bat '''
                sf force:auth:jwt:grant ^
                --client-id %SF_CONSUMER_KEY% ^
                --jwt-key-file server.key ^
                --username %SF_USERNAME% ^
                --instance-url https://login.salesforce.com ^
                --set-default-dev-hub ^
                --alias projectdemosfdc
                '''
            }
        }

        stage('Deploy to Org') {
            steps {
                bat '''
                sf project deploy start ^
                --source-dir force-app ^
                --target-org projectdemosfdc ^
                --wait 10
                '''
            }
        }

        stage('Run Tests') {
            steps {
                bat '''
                sf apex run test ^
                --target-org projectdemosfdc ^
                --wait 10 ^
                --result-format human
                '''
            }
        }
    }
}