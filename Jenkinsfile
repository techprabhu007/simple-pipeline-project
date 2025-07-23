pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        AWS_ACCOUNT_ID = "778813324501"
        ECR_REPO = "simple-pipeline"
        IMAGE_TAG = "latest"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO} .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag ${ECR_REPO}:latest ${ECR_URI}'
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push ${ECR_URI}'
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@44.243.7.88
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com &&
                docker pull ${ECR_URI} &&
                docker stop nodeapp || true &&
                docker rm nodeapp || true &&
                docker run -d --name nodeapp -p 80:3000 ${ECR_URI}
                "
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up local Docker images to free space...'
            sh '''
            docker rmi ${ECR_URI} || true
            docker rmi ${ECR_REPO}:latest || true
            docker system prune -f || true
            '''
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}