pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-image-name'
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
                    def dockerImage = docker.build("${IMAGE_NAME}")
                    // Save image reference for later
                    env.DOCKER_IMAGE_NAME = dockerImage.imageName()
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
                            def dockerImage = docker.image("${IMAGE_NAME}")
                            dockerImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh 'ansible-playbook deploy.yml'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
