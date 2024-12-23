pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'healthCheckAWS'  
        AWS_REGION = 'us-east-1'
        ECR_REPOSITORY_URI = '699475921831.dkr.ecr.us-east-1.amazonaws.com/healthcheckaws'  
        EC2_INSTANCE_IP = '54.242.12.114'  
        EC2_SSH_KEY_PATH = '/home/ec2-user/Ec2.pem'
        EC2_USER = 'ec2-user'  
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Talfaza/Health-Check-AWS.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "$AWS_REGION") {
                    script {
                        sh '''
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY_URI
                        docker tag $DOCKER_IMAGE $ECR_REPOSITORY_URI:latest
                        docker push $ECR_REPOSITORY_URI:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    sh """
                    ssh -i $EC2_SSH_KEY_PATH $EC2_USER@$EC2_INSTANCE_IP 'docker pull $ECR_REPOSITORY_URI:latest && docker run -d --name my-container $ECR_REPOSITORY_URI:latest'
                    """
                }
            }
        }
    }
}
