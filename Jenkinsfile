pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '378131233092'
        ECR_REPO = 'contact-management-system'
        IMAGE_TAG = 'latest'
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                    docker tag ${ECR_REPO}:${IMAGE_TAG} ${IMAGE_URI}
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${IMAGE_URI}
                """
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@172.31.6.112 '
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 378131233092.dkr.ecr.us-east-1.amazonaws.com

                        docker pull 378131233092.dkr.ecr.us-east-1.amazonaws.com/contact-management-system:latest

                        docker stop contact-management-system || true

                        docker rm contact-management-system || true

                        docker run -d --name contact-management-system -p 80:80 --restart unless-stopped 378131233092.dkr.ecr.us-east-1.amazonaws.com/contact-management-system:latest
                        '
                    """
                }
            }
        }

    }

    post {
        success {
            echo 'Docker Image Successfully Built, Pushed and Deployed'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
