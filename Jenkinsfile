pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'  
        AWS_ACCOUNT_ID = '699475921831'  
        ECR_REPO = 'healthcheckaws'  
        IMAGE_NAME = 'healthcheckaws'  
        EC2_KEY_PATH = '/home/ec2-user/Ec2.pem'
        EC2_PUBLIC_IP= '54.242.12.114'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Talfaza/Health-Check-AWS.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build --no-cache -t $IMAGE_NAME .'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "$AWS_REGION") {
                    script {
                        def ecrLogin = sh(script: "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com", returnStdout: true).trim()

                        sh "docker tag $IMAGE_NAME:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_NAME"

                        sh "docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_NAME"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "$AWS_REGION") {
                    script {
                        sh '''
                            ssh -o StrictHostKeyChecking=no -i $EC2_KEY_PATH ec2-user@$EC2_PUBLIC_IP "docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_NAME && docker run -d $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_NAME"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
