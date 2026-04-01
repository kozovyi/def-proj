pipeline {
    agent any

    environment {
        IMAGE_NAME = 'kozovyi'
    }

    stages {
        stage('Prepare Jenkins Environment') {
            steps {
                echo 'Installing Ansible and SSH client inside Jenkins container...'
                sh 'apt-get update && apt-get install -y ansible openssh-client'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t \${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Registry') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker tag \${IMAGE_NAME}:latest $DOCKER_USER/\${IMAGE_NAME}:latest"
                    sh "docker push $DOCKER_USER/\${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Deploy with Ansible') {
            steps {
                echo 'Running Ansible playbook to deploy container...'
                dir('ansible') {
                    sh "ansible-playbook -i hosts.ini playbook.yml -e \"docker_image=$DOCKER_USER/\${IMAGE_NAME}:latest\""

                }
            }
        }
    }
}
