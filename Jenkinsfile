pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'chaitug/react-app:1.0'
        EKS_CLUSTER_NAME = 'my-cluster'
        AWS_REGION = 'us-west-2'
        GIT_REPO = 'https://github.com/Chaitanyagude/react-app.git'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', url: GIT_REPO
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        
        stage('Test Locally') {
            steps {
                sh 'docker run -d -p 3000:3000 $DOCKER_IMAGE'
            }
        }
        
        stage('Set Up Amazon EKS Cluster') {
            steps {
                script {
                    sh 'aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER_NAME'
                }
            }
        }
        
        stage('Install eksctl') {
            steps {
                sh 'curl --silent –location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp'
                sh 'sudo mv /tmp/eksctl /usr/local/bin'
                sh 'eksctl version'
            }
        }
        
        stage('Create EKS Cluster') {
            steps {
                sh 'eksctl create cluster --name $EKS_CLUSTER_NAME --version 1.29 --region $AWS_REGION --node-type t2.micro --nodes 2 --managed'
            }
        }
        
        stage('Install kubectl') {
            steps {
                sh 'curl -o kubectl https://amazon-eks.s3.$AWS_REGION.amazonaws.com/1.22.1/2022-01-06/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl'
                sh 'sudo mv ./kubectl /usr/local/bin'
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
                sh 'kubectl get svc react-todo-list'
            }
        }
    }
}
