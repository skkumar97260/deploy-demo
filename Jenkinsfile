pipeline {
    agent any
    
    tools { 
        nodejs "NodeJS" // Ensure NodeJS is configured in Jenkins Global Tool Configuration
    }
    
    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image" // Replace with your Docker Hub repo
        DOCKER_TAG = "latest"
        DOCKER_CREDENTIALS = 'dockerhub-credentials' // Replace with your Jenkins credentials ID
    }

    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/main']], 
                        userRemoteConfigs: [[credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/skkumar97260/deploy-demo.git']])
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
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
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --name <cluster-name> --region <region>' // Update with your cluster details
                    sh 'kubectl apply -f ./k8s/frontend-deployment.yaml'
                    sh 'kubectl rollout status deployment/nodejs-app' // Optional: Ensure deployment is successful
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
