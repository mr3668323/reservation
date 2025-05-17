pipeline {
    agent any

    environment {
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = 'ec2-15-207-20-13.ap-south-1.compute.amazonaws.com'
        REMOTE_DIR  = '/home/ubuntu/restaurant'
        SSH_KEY_ID  = '24ae6725-79f4-46a0-a300-72aec761e5c8'
        GIT_REPO    = 'https://github.com/mr3668323/reservation.git'
        BRANCH      = 'main'
    }

    stages {
        stage('Pull latest code') {
            steps {
                git credentialsId: '21e6d9ba-dbc2-4afe-8516-c7158eec8818', url: "${env.GIT_REPO}", branch: "${env.BRANCH}"
            }
        }

        stage('Upload to EC2') {
            steps {
                sshagent (credentials: ["${env.SSH_KEY_ID}"]) {
                    sh """
                    rsync -avz --delete -e "ssh -o StrictHostKeyChecking=no" ./ ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        stage('Run Docker Compose on EC2') {
            steps {
                sshagent (credentials: ["${env.SSH_KEY_ID}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                      cd ${REMOTE_DIR} &&
                      docker compose down &&
                      docker compose up -d --build
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ App deployed successfully on EC2.'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
