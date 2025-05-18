pipeline {
    agent any

    environment {
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = 'ec2-3-110-219-235.ap-south-1.compute.amazonaws.com'
        REMOTE_DIR  = '/home/ubuntu/restaurant1'
        SSH_KEY_ID  = '24ae6725-79f4-46a0-a300-72aec761e5c8'
        GIT_REPO    = 'https://github.com/mr3668323/reservation.git'
        BRANCH      = 'main'
    }

    stages {
        stage('Pull latest code') {
            steps {
                echo '📥 Cloning the Git repository...'
                git credentialsId: '21e6d9ba-dbc2-4afe-8516-c7158eec8818', url: "${env.GIT_REPO}", branch: "${env.BRANCH}"
            }
        }

        stage('Upload to EC2') {
            steps {
                echo '📤 Uploading code to EC2 instance...'
                sshagent (credentials: ["${env.SSH_KEY_ID}"]) {
                    sh """
                        rsync -avz --delete -e "ssh -o StrictHostKeyChecking=no" ./ ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        stage('Run Docker Compose on EC2') {
            steps {
                echo '🐳 Running Docker Compose on EC2...'
                sshagent (credentials: ["${env.SSH_KEY_ID}"]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'ENDSSH'
set -e
cd ${REMOTE_DIR}
echo "🔻 Stopping old containers..."
docker compose down --remove-orphans || echo "⚠️ Nothing to stop or error occurred"

echo "🚀 Starting new containers..."
docker compose up -d --build || { echo "❌ Docker Compose failed"; exit 1; }

echo "📦 Checking running containers..."
docker ps

echo "🌐 Checking if frontend is running on port 3000..."
sudo lsof -i :3000 || echo "⚠️ Port 3000 not in use"
ENDSSH
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ App deployed successfully on EC2. Visit http://3.110.219.235:3000'
        }
        failure {
            echo '❌ Deployment failed. Please check the logs above.'
        }
    }
}
