pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-image-name'
        DOCKER_NAMESPACE = 'kundlasandhyarani'
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
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
                    credentialsId: 'ec2-ssh-key', // Replace with your actual Jenkins SSH credential ID
                    keyFileVariable: 'KEYFILE'
                )]) {
                    script {
                        docker.image('willhallonline/ansible:latest').inside("--user root -v ${KEYFILE}:/root/.ssh/id_rsa:ro") {
                    sh '''
                        mkdir -p ~/.ssh
                        ssh-keyscan 3.106.215.254 >> ~/.ssh/known_hosts
                        cd ansible
                        export ANSIBLE_HOST_KEY_CHECKING=False
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
