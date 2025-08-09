pipeline {
    agent { label 'label1' }

    environment {
        GIT_REPO = 'https://github.com/shivanitejasree/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'shivani/publicrepo'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/a4s1i1l8/shivani/publicrepo'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '071493677835'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${BUILD_NUMBER}"
	EKS_CLUSTER = 'my-eks-cluster'
    }

    stages {
        stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e
                        echo "Installing AWS CLI..."
                        sudo apt update && sudo apt install -y unzip curl

                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        rm -rf aws
                        unzip -q awscliv2.zip
                        sudo ./aws/install --update
                        aws --version
                    '''
                }
            }
        }
        
	stage('Configure AWS Credentials') {
    steps {
        script {
            withCredentials([string(credentialsId: 'AWS_Access_Token', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_Secret', variable: 'AWS_SECRET_ACCESS_KEY')]){
                             sh '''
            echo Setting up AWS credentials for Jenkins...
  	    mkdir -p ~/.aws
  	    echo "[default]" > ~/.aws/credentials
            echo "aws_access_key_id=AKIARBJK26MF7IOOW6X2" >> ~/.aws/credentials
            echo "aws_secret_access_key=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" >> ~/.aws/credentials
            '''
                        }
           
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
                    withCredentials([string(credentialsId: 'AWS_Access_Token', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_Secret', variable: 'AWS_SECRET_ACCESS_KEY')]){
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr-public get-login-password --region ap-south-1 | docker login --username AWS --password-stdin public.ecr.aws/a4s1i1l8
                    '''
                        }
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
