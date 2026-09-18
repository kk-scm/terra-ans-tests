pipeline {
    agent any

    environment { 
        AWS_REGION = 'us-east-2' 
        ECR_REPOSITORY = 'jenkins-demo-repo' 
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('ECR Setup') {
            steps {
                script { 
                    def accountId = sh( 
                        script: 'aws sts get-caller-identity --query Account --output text', 
                        returnStdout: true 
                    ).trim() 
                    env.ECR_URL = "${accountId}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}" 
                    env.IMAGE_TAG = sh(
                                        script: 'git rev-parse --short HEAD',
                                        returnStdout: true
                                        ).trim()
                    sh ''' 
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL 
                    '''
                }
            }
        }

        stage('Docker Test') {
            steps {
                sh 'id'
                sh 'docker ps'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $ECR_URL:$IMAGE_TAG .'
            }
        }

        stage('Docker Push') { 
            steps { 
                sh 'docker push $ECR_URL:$IMAGE_TAG' 
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    ssh -i /home/ec2-user/.ssh/jenkins-ansible-key \
                    -o StrictHostKeyChecking=no \
                    ec2-user@3.145.46.44 \
                    "cd ~/terra-ans-tests/ && ansible-playbook -i ~/inventory/aws_ec2.yml deploy.yml -e 'image_tag=$IMAGE_TAG'"
                '''
            }
        }
   
    }
}