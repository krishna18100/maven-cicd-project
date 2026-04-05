pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_SERVER_IP   = "<YOUR_APP_SERVER_PRIVATE_IP>"
        APP_SERVER_USER = "ubuntu"
        APP_SERVER_DIR  = "/home/ubuntu/app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to App Server') {
            when { branch 'master' }
            steps {
                sshagent(['app-server-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER_USER}@${APP_SERVER_IP} \
                            'mkdir -p ${APP_SERVER_DIR}'
                        scp -o StrictHostKeyChecking=no target/*.jar \
                            ${APP_SERVER_USER}@${APP_SERVER_IP}:${APP_SERVER_DIR}/app.jar
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER_USER}@${APP_SERVER_IP} \
                            'pkill -f app.jar || true'
                    """
                }
            }
        }
    }

    post {
        success { echo "✅ Pipeline completed successfully!" }
        failure { echo "❌ Pipeline failed." }
    }
}
