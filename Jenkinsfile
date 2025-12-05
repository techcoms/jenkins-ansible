pipeline {
    agent any

    environment {
        APP_NAME   = "myapp"
        IMAGE_NAME = "yourdockerhubusername/myapp"
        TAG        = "latest"
    }

    stages {

        stage('Git Clone') {
            steps {
                git 'https://github.com/your-repo/your-project.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {

                    sh '''
                       echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                       docker push ${IMAGE_NAME}:${TAG}
                       docker logout
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ssh-key',
                                                  keyFileVariable: 'SSH_KEY')]) {

                    sh '''
                        ansible-playbook -i ansible/inventory ansible/deploy.yml \
                        --extra-vars "image=${IMAGE_NAME}:${TAG} app_name=${APP_NAME}" \
                        --private-key $SSH_KEY
                    '''
                }
            }
        }
    }
}
