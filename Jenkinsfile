pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_SERVER_IP = '172.31.6.102'
        APP_USER      = 'ubuntu'
        REMOTE_DIR    = '/opt/app'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: env.BRANCH_NAME,
                    credentialsId: 'github-creds',
                    url: 'https://github.com/saitejareddy01/jenkins-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to App Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/*.jar ${APP_USER}@${APP_SERVER_IP}:${REMOTE_DIR}/myapp.jar
                        ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER_IP} 'bash /opt/app/restart.sh'
                    """
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure  { echo 'Pipeline failed!' }
    }
}
