pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'akshitr801/ci-cd:latest'
        EC2_HOST = '3.21.163.11'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST << EOF
                      docker pull $DOCKER_IMAGE
                      docker stop app || true
                      docker rm app || true
                      docker run -d -p 3000:3000 --name app $DOCKER_IMAGE
                    EOF
                    '''
                }
            }
        }
    }
}
