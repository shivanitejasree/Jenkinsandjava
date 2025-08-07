pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/shivanitejasree/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'shivanitejasree/myrepo'
        ECR_PUBLIC_REPO_URI = '071493677835.dkr.ecr.ap-south-1.amazonaws.com/shivanitejasree/myrepo'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '071493677835'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
       stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e
                        echo "Installing AWS CLI..."
                        yum update
                        curl https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip
                        unzip -q awscliv2.zip
                        ./aws/install --update
                    '''
                }
            }
        }

             stage('Configure AWS Credentials') {
    steps {
        script {
            sh '''
            echo "Setting up AWS credentials for Jenkins..."
            mkdir -p /var/lib/jenkins/.aws
            echo "[default]" > /var/lib/jenkins/.aws/credentials
            echo "aws_access_key_id=AKIAWN26KB25BRBD26NF" >> /var/lib/jenkins/.aws/credentials
            echo "aws_secret_access_key=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" >> /var/lib/jenkins/.aws/credentials
            chown -R jenkins:jenkins /var/lib/jenkins/.aws
            '''
        }
    }
}


        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        echo "Building Java application..."
                        mvn clean -B -Denforcer.skip=true package
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr-public get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 071493677835.dkr.ecr.ap-south-1.amazonaws.com/shivanitejasree/myrepo
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        docker push ${IMAGE_URI}
                    '''
                }
            }
        }

    }
    
    post {
        success {
            echo "Docker image pushed to ECR successfully and deployed."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
