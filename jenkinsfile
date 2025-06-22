pipeline {
    agent any
    environment {
        DEPLOY_SERVER = '10.121.8.24'  // Use IP to avoid DNS issues
        DEPLOY_USER = 'jelastic'     // Replace with your actual username from ssh-credentials
        DEPLOY_PATH = '/var/www/webroot/ROOT'
        SSH_PORT = '22'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/ChandanKunwar/Test.git', branch: 'master'
            }
        }
        stage('Test Network and SSH') {
            steps {
                sshagent(credentials: ['ssh-credentials']) {
                    sh '''
                        echo "Testing ping to ${DEPLOY_SERVER}"
                        ping -c 4 ${DEPLOY_SERVER} || echo "Ping failed, check network"
                        echo "Testing SSH connection to ${DEPLOY_USER}@${DEPLOY_SERVER} on port ${SSH_PORT}"
                        ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no -o ConnectTimeout=30 ${DEPLOY_USER}@${DEPLOY_SERVER} "echo Connection successful" || exit 1
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials: ['ssh-credentials']) {
                    sh '''
                        ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} "rm -rf ${DEPLOY_PATH}/*"
                        scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r * ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
