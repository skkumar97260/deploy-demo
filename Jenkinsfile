pipeline {
    agent any
                         
    tools {
        nodejs "nodejs" // Ensure Node.js is configured in Jenkins
    }

    environment {
        DOCKER_IMAGE = "skkumar97260/sk-image"
        DOCKER_TAG = "latest"
        AWS_CLUSTER_NAME = "my-eks-cluster1"
        AWS_REGION = "us-east-1"
        KUBERNETES_NAMESPACE = "nodejs-app-namespace"
    }

    stages {
        stage('Clone Code from GitHub') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        credentialsId: 'GITHUB_CREDENTIALS',
                        url: 'https://github.com/skkumar97260/deploy-demo.git'
                    ]]
                )
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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([ 
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${AWS_CLUSTER_NAME}

                        echo "Creating Kubernetes namespace..."
                        kubectl create namespace ${KUBERNETES_NAMESPACE} || true

                        echo "Applying Kubernetes manifests..."
                        kubectl apply -n ${KUBERNETES_NAMESPACE} -f k8s/deployment.yaml
                        kubectl apply -n ${KUBERNETES_NAMESPACE} -f k8s/service.yaml
                        kubectl apply -n ${KUBERNETES_NAMESPACE} -f k8s/ingress.yaml
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
