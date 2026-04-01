pipeline {
    agent any

    environment {
        DOCKER_CREDS_ID = 'docker-hub'
        HOST_SSH_CREDS_ID = 'ssh-host-vagrant'
        IMAGE_NAME = 'kozovyi'
        DOCKER_HOST_IP = '172.17.0.1' 
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Registry') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker tag ${IMAGE_NAME}:latest ${DOCKER_USER}/${IMAGE_NAME}:latest
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy via Host Ansible') {
            steps {
                echo 'Connecting to host to run Ansible...'
                sshagent(["${HOST_SSH_CREDS_ID}"]) {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no vagrant@${DOCKER_HOST_IP} \
                            'cd /home/vagrant/shared-terraform/ansible-lab-11 && \
                             /home/vagrant/.local/bin/ansible-playbook -i inventory/hosts.ini playbook.yml \
                             -e "docker_image=${DOCKER_USER}/${IMAGE_NAME}:latest" \
                             -e "docker_user=${DOCKER_USER}" \
                             -e "docker_pass=${DOCKER_PASS}"'
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            sh 'docker logout'
        }
    }
}