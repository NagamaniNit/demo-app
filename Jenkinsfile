pipeline {

    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "123456789012"
        IMAGE_NAME = "cbci-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Tag Image') {
            steps {
                sh """
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                docker push \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                sed -i 's/latest/${IMAGE_TAG}/g' deployment.yaml
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
