pipeline {
    agent any

    parameters {
        string(name: 'REMOTE_HOST', defaultValue: '192.168.56.101', description: 'IP address of the Target VM')
        string(name: 'REMOTE_USER', defaultValue: 'bohelskyi', description: 'SSH user for the Target VM')
        string(name: 'SSH_CRED_ID', defaultValue: 'target-server-ssh-key', description: 'Jenkins Credentials ID for SSH key')
    }

    stages {
        stage('Environment Setup') {
            steps {
                script {
                    echo "Preparing to deploy to ${params.REMOTE_USER}@${params.REMOTE_HOST}"
                }
            }
        }

        stage('Infrastructure Management') {
            steps {
                // Використовуємо плагін SSH Agent для безпечного керування ключами
                sshagent([params.SSH_CRED_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${params.REMOTE_USER}@${params.REMOTE_HOST} '
                            echo "--- System Health Check ---"
                            uptime
                            
                            echo "--- Ensuring Apache2 Installation ---"
                            sudo apt-get update
                            sudo apt-get install -f -y
                            sudo apt-get install -y apache2
                            
                            echo "--- Service Management ---"
                            sudo systemctl enable apache2
                            sudo systemctl start apache2
                        '
                    """
                }
            }
        }

        stage('Quality Assurance (QA)') {
            steps {
                echo "Running Health Check..."
                sh "curl -sI http://${params.REMOTE_HOST} | grep HTTP"
            }
        }

        stage('Log Analysis & Monitoring') {
            steps {
                sshagent([params.SSH_CRED_ID]) {
                    sh """
                        ssh ${params.REMOTE_USER}@${params.REMOTE_HOST} "
                            if sudo test -f /var/log/apache2/access.log; then
                                echo '--- Analyzing Security/Error Logs ---'
                                sudo grep -E ' [45][0-9][0-9] ' /var/log/apache2/access.log || echo 'Clean: No 4xx/5xx errors.'
                            else
                                echo 'Error: Critical log file missing!'
                                exit 1
                            fi
                        "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful on ${params.REMOTE_HOST}!"
        }
        failure {
            echo "Deployment Failed! Please check the logs above."
        }
        always {
            cleanWs() // Очищення робочої директорії після завершення
        }
    }
}