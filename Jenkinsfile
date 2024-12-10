pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "your-dockerhub-username"
        IMAGE_NAME = "nodejs-app"
        KUBE_CONFIG = credentials('kubeconfig-id') // Replace with Jenkins kubeconfig credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo-url.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials-id') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig-id']) {
                        sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl set image deployment/nodejs-app nodejs-app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            mail to: 'your-email@example.com',
                 subject: "Deployment Success: Build #${env.BUILD_NUMBER}",
                 body: "The build and deployment were successful."
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Deployment Failed: Build #${env.BUILD_NUMBER}",
                 body: "The build or deployment failed. Check Jenkins for details."
        }
    }
}
