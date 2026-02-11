pipeline {
    agent any

    environment {
        // Jenkins connection – adjust these!
        JENKINS_URL = 'http://54.188.28.149:8080/'   // CHANGE
        ADMIN_USER = 'admin'                                 // CHANGE if different
        ADMIN_TOKEN = credentials('jenkins-admin-token')     // ID of your secret text credential

        // SMTP (Gmail app password) – adjust!
        SMTP_CREDS = credentials('smtp-creds')
        SMTP_USER = "${SMTP_CREDS_USR}"
        SMTP_PASSWORD = "${SMTP_CREDS_PSW}"
        SMTP_SERVER = 'smtp.gmail.com'
        SMTP_PORT = '587'
        FROM_EMAIL = "${SMTP_USER}"

        // CSV file (in repository root)
        CSV_PATH = 'users.csv'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Using CSV from workspace: ${CSV_PATH}"
            }
        }

        stage('Run Provisioning in Docker') {
            agent {
                docker {
                    image 'python:3-slim'
                    reuseNode true   // use the same workspace
                }
            }
            steps {
                sh '''
                    # Install only the required Python packages
                    pip install --no-cache-dir requests urllib3

                    # Execute the provisioning script
                    python provision_jenkins_users.py
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
                to: 'aditya.tripathi998@gmail.com'   // CHANGE to your email/team
            )
        }
    }
}
