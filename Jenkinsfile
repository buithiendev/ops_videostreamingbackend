pipeline {
    agent any
    environment {
        ASK_DEV_FILE = 'deploy-dev.yaml'
        AKS_STAGING_FILE = 'deploy-staging.yaml'
        DOCKER_COMPOSE_FILE = 'docker-compose.yaml'
        NAME_ACR = 'ACRAssignmentCloud001'
        NAME_AKS = 'AKS001'
        URL_ACR = 'http://acrassignmentcloud001.azurecr.io'
        URL_ACR_NO_HTTP = 'acrassignmentcloud001.azurecr.io'
        URL_REPOSITORY = 'https://buithiendev@bitbucket.org/buithiendev/videostreaming_backend.git'
        URL_AKS = 'http://aks001-dns-iwlpbu00.hcp.japaneast.azmk8s.io'

        NAME_RS = 'AssignmentCloud'
        NAME_IMAGE_BACKEND = 'nestjsbackend'
        NAME_CONTAINER_BACKEND = 'nestjs-backend-container'

        CREDENTIAL_REPOSITORY = 'CREDENTIAL_REPO'
        CREDENTIAL_AKS = 'CREDENTIAL_AKS'
        CREDENTIAL_ACR = 'CREDENTIAL_ACR'
        AZURE_SP_APP_ID = credentials('AZURE_SP_APP_ID')
        AZURE_SP_PASSWORD = credentials('AZURE_SP_PASSWORD')
        AZURE_SP_TENANT = credentials('AZURE_SP_TENANT')
    }

    tools {
        nodejs "node"
    }

    stages { 
        stage('Check tools') {
            steps {
                echo "Check tools"
                sh "node -v"
                sh "git version"
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: CREDENTIAL_REPOSITORY, url: URL_REPOSITORY]])
            }
        }

        stage('Build with NodeJS') {
            steps {
                echo "Build with NodeJS"
                script {
                    sh 'npm install'
                }
            }
        }

        stage('Run tests') {
            steps {
                echo "Run tests"
                script {
                    sh 'npm test'
                }
            }
        }
       
        stage('Upload Image to ACR') {
            steps{   
                echo "Upload Image to ACR"
                script {  
                    withDockerRegistry([credentialsId: CREDENTIAL_ACR, url: URL_ACR]) {
                        sh "docker compose -f $DOCKER_COMPOSE_FILE build"
                        sh "docker tag $NAME_IMAGE_BACKEND $URL_ACR_NO_HTTP/$NAME_IMAGE_BACKEND:latest"
                        sh "docker push $URL_ACR_NO_HTTP/$NAME_IMAGE_BACKEND:latest"
                    }
                }
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
                    sh 'chmod +x ./kubectl'
                    sh 'mv ./kubectl /usr/local/bin/kubectl'
                    sh 'curl -sL https://aka.ms/InstallAzureCLIDeb |bash'
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo "Deploy to Dev"
                withKubeConfig(credentialsId: CREDENTIAL_AKS, serverUrl: URL_AKS) {
                    sh 'az login --service-principal --username $AZURE_SP_APP_ID --password $AZURE_SP_PASSWORD --tenant $AZURE_SP_TENANT --output none'
                    sh "az aks get-credentials --resource-group $NAME_RS --name $NAME_AKS"
                    sh "kubectl apply -f $ASK_DEV_FILE"
                }
            }
        }

        stage('Clean') {
            steps {
                echo "Clean"
                script {
                    sh 'docker compose down --volumes --rmi all'
                }
            }
        }
    }
}
