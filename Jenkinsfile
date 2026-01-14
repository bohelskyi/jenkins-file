pipeline {
    agent any
    environment {
        REMOTE_HOST = '192.168.56.101' 
        REMOTE_USER = 'bohelskyi'
    }

    stages {
        stage('Check Connectivity') {
            steps {
                echo "Checking connection to ${REMOTE_HOST}..."
                sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'uptime'"
            }
        }

        stage('Install Apache2') {
            steps {
                echo "Installing Apache on remote machine..."
                sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} '
                        sudo apt-get update && \
                        sudo apt-get install -y apache2 && \
                        sudo systemctl enable apache2 && \
                        sudo systemctl start apache2
                    '
                """
            }
        }
        
        stage('Verify Web Server') {
            steps {
                echo "Verifying installation..."
                sh "curl -sI http://${REMOTE_HOST} | grep HTTP"
            }
        }
        
        stage('Check Apache Logs') {
            steps {
                echo "Analyzing access logs for 4xx and 5xx errors..."
                sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} "
                        if sudo test -f /var/log/apache2/access.log; then
                            echo '--- Found Error Logs ---'
                            sudo grep -E ' [45][0-9][0-9] ' /var/log/apache2/access.log || echo 'No 4xx or 5xx errors found.'
                        else
                            echo 'Log file not found or inaccessible!'
                            exit 1
                        fi
                    "
                """
            }
        }
    }
}