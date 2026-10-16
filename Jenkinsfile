
pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        IMAGE_NAME = 'devops-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rahulrajdsce/devops-project.git'
            }
        }

        stage('Validate Application') {
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci --omit=dev
                    node --check server.js
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Load Image into Minikube') {
            steps {
                sh '''
                    minikube image load ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/devops-app \
                      devops-app=${IMAGE_NAME}:${BUILD_NUMBER}

                    kubectl rollout status deployment/devops-app \
                      --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments devops-app
                    kubectl get pods -l app=devops-app
                    kubectl get services devops-app-service
                '''
            }
        }
    }
}
