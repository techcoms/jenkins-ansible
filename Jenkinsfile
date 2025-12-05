pipeline {
    agent any

    environment {
        APP_NAME   = "myapp"
        DOCKER_USER = credentials('dockerhub-creds')  // if using token-style creds
        IMAGE_NAME = "techcoms/myapp"
        TAG        = "latest"
    }

    stages {

        stage('Clone repository') {
            steps {
                git branch: 'main', url: 'https://github.com/techcoms/jenkins-ansible.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {

                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'deploy-ssh-key',
                                    keyFileVariable: 'SSH_KEY')
                ]) {
                    sh """
                        ansible-playbook -i ansible/inventory ansible/deploy.yml \
                        --extra-vars "image=${IMAGE_NAME}:${TAG} app_name=${APP_NAME}" \
                        --private-key $SSH_KEY
                    """
                }
            }
        }
    }
}
