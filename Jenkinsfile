pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ci-cd-demo'
        DOCKER_NAMESPACE = 'kundlasandhyarani'
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_NAMESPACE}/${IMAGE_NAME}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    script {
                        docker.withRegistry(DOCKER_REGISTRY, 'dockerhub-creds') {
                            docker.image("${DOCKER_NAMESPACE}/${IMAGE_NAME}").push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key',
                    keyFileVariable: 'KEYFILE'
                )]) {
                    script {
                        docker.image('willhallonline/ansible:latest').inside("--user root -v ${KEYFILE}:/tmp/id_rsa:ro") {
                            sh '''
                                mkdir -p ~/.ssh
                                cp /tmp/id_rsa ~/.ssh/id_rsa
                                chmod 600 ~/.ssh/id_rsa
                                ssh-keyscan 16.176.228.219 >> ~/.ssh/known_hosts

                                cd ansible
                                ansible-playbook -i inventory.ini deploy.yml
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment pipeline completed successfully!'
        }
        failure {
            echo '❌ Deployment pipeline failed. Please check the logs above.'
        }
    }
}
