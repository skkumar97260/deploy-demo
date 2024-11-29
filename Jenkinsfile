pipeline {
    agent any
    
    tools { nodejs "node" }
    
    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    checkout scmGit(
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[credentialsId: 'devopshintdocker', url: 'https://github.com/skkumar97260/deploy-demo']]
                    )
                }
            }
        }
        
        stage('Node JS Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build Node JS Docker Image') {
            steps {
                script {
                    // Use your Docker image name
                    sh 'docker build -t skkumar97260/sk-image .'
                }
            }
        }
        
        stage('Deploy Docker Image to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'devopshintdocker', variable: 'devopshintdocker')]) {
                        // Authenticate DockerHub
                        sh 'docker login -u skkumar97260 -p ${devopshintdocker}'
                    }
                    // Push the image to DockerHub
                    sh 'docker push skkumar97260/sk-image'
                }
            }
        }
        
        stage('Deploying Node App to Kubernetes') {
            steps {
                script {
                    // Update kubeconfig for EKS cluster
                    sh 'aws eks update-kubeconfig --name sample --region ap-south-1'
                    // Check Kubernetes namespaces
                    sh 'kubectl get ns'
                    // Apply the Kubernetes manifest (nodejsapp.yaml)
                    sh 'kubectl apply -f nodejsapp.yaml'
                }
            }
        }
    }
}
