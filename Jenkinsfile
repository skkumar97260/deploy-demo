pipeline {
    agent any

    tools {
        nodejs "nodejs" // Ensure this matches the name of your Node.js installation in Jenkins
    }

    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image"
        DOCKER_TAG = "latest"
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
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
                withCredentials([string(credentialsId: 'dockerhub-credentials', variable: 'DOCKER_TOKEN')]) {
                    script {
                        // Securely login using password-stdin
                        sh """echo \$DOCKER_TOKEN | docker login -u skkumar97260 --password-stdin"""
                        // Push the image
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "aws eks update-kubeconfig --name ${AWS_CLUSTER_NAME} --region ${AWS_REGION}"
                sh "kubectl apply -f nodejsapp.yaml"
                sh "kubectl rollout status deployment/nodejs-app"
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
