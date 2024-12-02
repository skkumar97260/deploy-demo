pipeline {
    agent any
 
    tools {
        nodejs "nodejs" // Ensure Node.js is configured in Jenkins tools
    }

    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image"
        DOCKER_TAG = "latest"
        AWS_CLUSTER_NAME = "My-eks-cluster"
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        echo "Logging into DockerHub"
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        '''
                        echo "Pushing Docker image to DockerHub"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        echo "Updating kubeconfig for EKS"
                        sh '''
                            export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                            export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${AWS_CLUSTER_NAME}
                        '''
                        echo "Deploying application to Kubernetes"
                        sh '''
                            kubectl apply -f deployment.yaml
                            kubectl rollout status deployment/nodejs-app
                        '''
                    }
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
