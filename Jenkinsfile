pipeline {
    agent any

    environment {
        AWS_REGION      = 'ap-south-1'
        ECR_REGISTRY    = '123456789.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPO        = 'cicd-devops-project'
        IMAGE_TAG       = "v${BUILD_NUMBER}"
        EC2_HOST        = 'YOUR_EC2_PUBLIC_IP'
        EC2_USER        = 'ubuntu'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                echo '☁️ Pushing image to AWS ECR...'
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                        docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '🚀 Deploying to EC2...'
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        
                        docker stop webapp || true
                        docker rm webapp   || true
                        
                        docker pull ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                        docker run -d --name webapp -p 80:80 ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success { echo '✅ Pipeline succeeded! App is live.' }
        failure { echo '❌ Pipeline failed. Check logs above.' }
    }
}
