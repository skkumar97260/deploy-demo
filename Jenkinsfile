pipeline {
    agent any
    
    tools {
        nodejs "nodejs" // Use the correct name of the NodeJS installation in Jenkins
    }
    
    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image" // Docker Hub repository name
        DOCKER_TAG = "latest" // Docker tag
        DOCKER_CREDENTIALS = 'dockerhub-credentials' // Jenkins credentials ID for Docker Hub
        AWS_CLUSTER_NAME = "sample" // Replace with your AWS EKS cluster name
        AWS_REGION = "us-east-1" // Replace with your AWS region
    }

    stages {
        stage("Clone Code from GitHub") {
            steps {
                script {
                    checkout scmGit(
                        branches: [[name: '*/main']], 
                        userRemoteConfigs: [[credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/skkumar97260/deploy-demo.git']]
                    )
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install --prefix ./frontend'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ./frontend"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub-credentials', variable: 'DOCKER_TOKEN')]) {
                        sh "echo $DOCKER_TOKEN | docker login -u <your-dockerhub-username> --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Configure kubectl to connect to the AWS EKS cluster
                    sh "aws eks update-kubeconfig --name ${AWS_CLUSTER_NAME} --region ${AWS_REGION}"
                    
                    // Apply Kubernetes deployment and service YAML files
                    sh 'kubectl apply -f ./k8s/frontend-deployment.yaml'
                    
                    // Optional: Wait for the deployment to complete
                    sh 'kubectl rollout status deployment/nodejs-app'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution finished.'
        }
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
