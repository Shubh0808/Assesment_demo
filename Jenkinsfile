pipeline {
    agent any

    environment {
        ACR_NAME = "demoregistry1234"
        IMAGE_NAME = "python-backend"
        AZURE_CREDENTIALS = credentials('azure-sp') 
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Shubh0808/Assesment_demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $ACR_NAME.azurecr.io/$IMAGE_NAME:latest ./app"
                }
            }
        }

        stage('Login to ACR') {
            steps {
                script {
                    sh "az login --service-principal -u ${AZURE_CREDENTIALS_USR} -p ${AZURE_CREDENTIALS_PSW} --tenant <tenant-id>"
                    sh "az acr login --name $ACR_NAME"
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                script {
                    sh "docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:latest"
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    sh "az aks get-credentials --resource-group demo-rg --name demo-aks"
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
