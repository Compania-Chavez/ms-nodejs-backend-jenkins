pipeline {
    agent {
        docker { image 'devops-agent:latest' }
    }

    environment {
        APELLIDO = "chavez"
        SHORT_SHA = "${env.GIT_COMMIT[0..6]}"
        IMAGE_NAME = "acr${APELLIDO}.azurecr.io/my-nodejs-app"
        TAG = "${SHORT_SHA}"
        IMAGE = "${IMAGE_NAME}:${TAG}"
        RESOURCE_GROUP = "rg-cicd-terraform-app-${APELLIDO}"
        ACR_NAME = "acr${APELLIDO}"
    }

    stages {

        stage('Azure Login') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-clientId',        variable: 'AZ_CLIENT_ID'),
                    string(credentialsId: 'azure-clientSecret',   variable: 'AZ_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenantId',       variable: 'AZ_TENANT_ID'),
                    string(credentialsId: 'azure-subscriptionId', variable: 'AZ_SUBSCRIPTION_ID')
                ]) {
                    sh '''
                      echo ">>> Azure login..."
                      az login --service-principal \
                        --username $AZ_CLIENT_ID \
                        --password $AZ_CLIENT_SECRET \
                        --tenant $AZ_TENANT_ID
                      az account set --subscription $AZ_SUBSCRIPTION_ID
                      az account show
                    '''
                }
            }
        }
        
        stage('Instalar dependencias') {
            steps {
                sh '''
                  echo ">>> Instalar dependencias"
                  npm install
                '''
            }
        }

        
        stage('Pruebas unitarias') {
            steps {
                sh '''
                  echo ">>> Pruebas unitarias"
                  npm run test:unit
                '''
            }
        }

        
        stage('Pruebas de integración') {
            steps {
                sh '''
                  echo ">>> Pruebas de integración"
                  npm run test:integration
                '''
            }
        }
        
        stage('ACR Login') {
            steps {
                sh '''
                  echo ">>> ACR Login"
                  az acr login --name $ACR_NAME
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh '''
                  echo ">>> Construyendo imagen $IMAGE"
                  docker build -t $IMAGE .
                  docker push $IMAGE
                '''
            }
        }
        
        stage('Deploy to DEV') {
            steps {
                script {
                    env.ENV = "dev"
                    env.API_PROVIDER_URL = "http://dev.api.com"
                    env.APP_NAME = "aca-env-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Configurando ACR credentials para Container App..."
                  ACR_SERVER=$(az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query loginServer --output tsv)
                  echo "Servidor ACR: $ACR_SERVER"
                  az containerapp registry set \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --server $ACR_SERVER \
                    --identity system

                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('Approval QA') {
            steps {
                input message: "Aprobar despliegue en QA?"
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    env.ENV = "qa"
                    env.API_PROVIDER_URL = "http://qa.api.com"
                    env.APP_NAME = "aca-env-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Configurando ACR credentials para Container App..."
                  ACR_SERVER=$(az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query loginServer --output tsv)
                  echo "Servidor ACR: $ACR_SERVER"
                  az containerapp registry set \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --server $ACR_SERVER \
                    --identity system

                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('Approval PRD') {
            steps {
                input message: "Aprobar despliegue en PRD?"
            }
        }

        stage('Deploy to PRD') {
            steps {
                script {
                    env.ENV = "prd"
                    env.API_PROVIDER_URL = "http://prd.api.com"
                    env.APP_NAME = "aca-env-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Configurando ACR credentials para Container App..."
                  ACR_SERVER=$(az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query loginServer --output tsv)
                  echo "Servidor ACR: $ACR_SERVER"
                  az containerapp registry set \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --server $ACR_SERVER \
                    --identity system

                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('Print Endpoint') {
            steps {
                sh '''
                  ENDPOINT=$(az containerapp show \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --query properties.configuration.ingress.fqdn -o tsv)
                  echo "Endpoint del Container App ($ENV): https://$ENDPOINT"
                '''
            }
        }
    }
}
