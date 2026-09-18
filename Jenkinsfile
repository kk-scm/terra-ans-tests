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
                    ssh ec2-user@<ANSIBLE_IP> \
                    "ansible-playbook -i ~/inventory/aws_ec2.yml ~/project/deploy-app.yml -e 'image_tag=$IMAGE_TAG'"
                '''
            }
        }
   
    }
}