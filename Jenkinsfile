pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubuser/node-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        LEMP_SERVER = "<SERVER-IP>"
        LEMP_USER = "ubuntu"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'github-credentials',
                url: 'https://github.com/<your-username>/<repo>.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['lemp-server-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${LEMP_USER}@${LEMP_SERVER} "
                    docker pull ${DOCKER_IMAGE}:latest &&
                    docker stop nodeapp || true &&
                    docker rm nodeapp || true &&
                    docker run -d --name nodeapp -p 80:3000 ${DOCKER_IMAGE}:latest
                    "
                    '''
                }
            }
        }
    }
}
