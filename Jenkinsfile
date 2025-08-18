pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-image-name'
        DOCKER_NAMESPACE = 'kundlasandhyarani' // Your Docker Hub username
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
                    def dockerImage = docker.build("${DOCKER_NAMESPACE}/${IMAGE_NAME}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', // Make sure this matches your Jenkins credentials ID
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    script {
                        docker.withRegistry(DOCKER_REGISTRY, 'dockerhub-creds') {
                            def dockerImage = docker.image("${DOCKER_NAMESPACE}/${IMAGE_NAME}")
                            dockerImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
    steps {
        script {
            docker.image('willhallonline/ansible:latest').inside {
                sh 'ansible-playbook deploy.yml'
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
}
