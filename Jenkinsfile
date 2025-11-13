pipeline {
    agent any

    environment {
        ACR_NAME = "demoregistry1234"
        IMAGE_NAME = "python-backend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Shubh0808/Assesment_demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${ACR_NAME}.azurecr.io/${IMAGE_NAME}:latest ./app
                """
            }
        }

        stage('Login to ACR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'azure-sp',
                        usernameVariable: 'AZURE_SP_USR',
                        passwordVariable: 'AZURE_SP_PSW'
                    )
                ]) {
                    sh """
                        az login --service-principal \
                            -u "${AZURE_SP_USR}" \
                            -p "${AZURE_SP_PSW}" \
                            --tenant 943ce175-eaca-4be1-a92c-7e492c94921d

                        az acr login --name ${ACR_NAME}
                    """
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                sh "docker push ${ACR_NAME}.azurecr.io/${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh """
                    az aks get-credentials --resource-group demo-rg --name demo-aks --overwrite-existing
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
