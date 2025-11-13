pipeline {
    agent any

    environment {
        ACR_NAME = "demoregistry1234"
        IMAGE_NAME = "python-backend"
        AZURE_SP = credentials('azure-sp')  
        TENANT_ID = "943ce175-eaca-4be1-a92c-7e492c94921d"   
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Shubh0808/Assesment_demo.git']]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $ACR_NAME.azurecr.io/$IMAGE_NAME:latest ./app
                    """
                }
            }
        }

        stage('Login to ACR') {
            steps {
                script {
                    sh """
                        az login --service-principal \
                          -u ${AZURE_SP_USR} \
                          -p ${AZURE_SP_PSW} \
                          --tenant $TENANT_ID

                        az acr login --name $ACR_NAME
                    """
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                script {
                    sh """
                        docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    sh """
                        az aks get-credentials --resource-group demo-rg --name demo-aks --overwrite-existing

                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
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
