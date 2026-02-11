pipeline {
    agent any

    environment {
        // Jenkins admin token (stored as secret text)
        ADMIN_TOKEN = credentials('jenkins-admin-token')
        ADMIN_USER = 'admin'   // the username that owns the token
        JENKINS_URL = 'http://54.188.28.149:8080/'   // CHANGE to your Jenkins URL

        // SMTP credentials (stored as username/password)
        SMTP_CREDS = credentials('smtp-creds')
        SMTP_USER = "${SMTP_CREDS_USR}"      // automatically populated by credentials binding
        SMTP_PASSWORD = "${SMTP_CREDS_PSW}"  // automatically populated
        SMTP_SERVER = 'smtp.gmail.com'
        SMTP_PORT = '587'
        FROM_EMAIL = 'aditya.tripathi998@gmail.com'   // same as SMTP_USER

        // CSV file in workspace
        CSV_PATH = 'users.csv'
    }

    stages {
        stage('Checkout') {
            steps {
                // If you store the CSV and script in SCM, use 'checkout scm'
                // For manual upload, skip or copy from workspace
                echo "Using CSV from workspace: ${CSV_PATH}"
            }
        }
        stage('Setup Python') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install requests urllib3
                '''
            }
        }
        stage('Provision Users') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 provision_jenkins_users.py
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'users.csv, provision_jenkins_users.py', fingerprint: true
        }
        failure {
            emailext(
                subject: "User Provisioning FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Check console: ${env.BUILD_URL}console",
                to: 'your.name@gmail.com'   // or your team email
            )
        }
    }
}
