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
                sh 'docker build -t $ECR_URL:$BUILD_NUMBER .'
            }
        }

        stage('Docker Push') { 
            steps { 
                sh 'docker push $ECR_URL:$BUILD_NUMBER' 
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run --rm $ECR_URL:$BUILD_NUMBER'
            }
        }

    }
}