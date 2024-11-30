pipeline {
    agent any
 
    tools {
        nodejs "nodejs" // Ensure this matches the name of your Node.js installation in Jenkins
    }

    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image"
        DOCKER_TAG = "latest"
        AWS_CLUSTER_NAME = "demo"
        AWS_REGION = "us-east-1"
    }

    stages {
        stage('Clone Code from GitHub') {
            steps {
                script {
                    checkout scmGit(
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/skkumar97260/deploy-demo.git']]
                    )
                }
            }
        }

        stage('Validate Node.js and npm') {
            steps {
                sh 'node -v && npm -v'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    if (fileExists('package.json')) {
                        sh 'npm install'
                    } else {
                        error "Missing package.json in the root directory"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                // Using username and password credentials
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        echo "Docker login using credentials"
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        '''
                        echo "Pushing Docker image"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
               script {
            // Set up AWS EKS credentials
            withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                // Set AWS credentials as environment variables for AWS CLI
                sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION=${AWS_REGION}
                    
                    # Update kubeconfig to use the EKS cluster
                    aws eks update-kubeconfig --name ${AWS_CLUSTER_NAME} --region ${AWS_REGION}
                    
                    # Debug: Check if kubectl is configured correctly
                    kubectl config view
                    kubectl cluster-info
                    kubectl get nodes
                '''
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
