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
                    // Build and tag the Docker image
                    dockerImage = docker.build("${IMAGE_NAME}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', // Replace with your Jenkins credential ID
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    script {
                        docker.withRegistry(DOCKER_REGISTRY, "${DOCKER_USERNAME}") {
                            dockerImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                // Replace with your actual Ansible command
                sh 'ansible-playbook deploy.yml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
