pipeline {
    agent {
        docker { image 'devops-agent:latest' }
    }

    environment {
        NODE_VERSION = "20"
        APELLIDO = "chavez" // Reemplazar por tu apellido
        SHORT_SHA = "${env.GIT_COMMIT[0..6]}"
        IMAGE_NAME = "acr${APELLIDO}.azurecr.io/my-nodejs-app"
        TAG = "${SHORT_SHA}"
        IMAGE = "${IMAGE_NAME}:${TAG}"
        RESOURCE_GROUP = "rg-cicd-terraform-app-${APELLIDO}"
        ACR_NAME = "acr${APELLIDO}"
    }

    stages {       
        stage('Hello world') {
            steps {
                script { 
                    // Declarar más variables de entorno
                    env.VARIABLE = "demo123"
                }
                // Primer step
                sh '''
                  echo ">>> Impresión Hello world"
                  echo "Hello world"
                  echo "Variable declarada en script: $VARIABLE"
                  echo "Variable declarada en environment: $APELLIDO"
                '''
                // Step adicional
                sh '''
                  echo ">>> Versiones instaladas:"
                  node -v
                  npm -v
                  docker --version
                  az version
                '''
            }
        }

        stage('[CI] Instalar dependencias') {
            steps {
                checkout scm
                // Instalar dependencias
                sh '''
                  npm install
                '''
            }
        }

        stage('[CI] Ejecutar pruebas unitarias') {
            steps {
                // Ejecutar pruebas unitarias
                sh '''
                  npm run test:unit
                '''
            }
        }

        stage('[CI] Ejecutar pruebas de integración') {
            steps {
                // Ejecutar pruebas de integración
                sh '''
                  npm run test:integration
                '''
            }
        }

        stage('[CI] Azure Login') {
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

        stage('[CI] Docker Login') {
            steps {
                // Login a ACR
                sh '''
                  az acr login --name $ACR_NAME
                '''
            }
        }

        stage('[CI] Build and Push Docker Image') {
            steps {
                // build image
                sh '''
                  echo "Construyendo imagen $IMAGE"
                  docker build -t $IMAGE .
                '''
                // push image
                sh '''
                  echo "Pushing imagen $IMAGE"
                  docker push $IMAGE
                '''
            }
        }
        
        stage('[CD][dev] Configurar ACR credentials para Container App') {
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
                '''
            }
        }

        stage('[CD][dev] Deploy a Azure Container App') {
            steps {
                script {
                    env.ENV = "dev"
                    env.API_PROVIDER_URL = "http://dev.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('[CD][dev] Imprimir endpoint del Container App') {
            steps {
                script {
                    env.ENV = "dev"
                    env.API_PROVIDER_URL = "http://dev.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
                sh '''
                  ENDPOINT=$(az containerapp show \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --query properties.configuration.ingress.fqdn -o tsv)
                  echo "Endpoint del Container App ($ENV): https://$ENDPOINT"
                '''
            }
        }

        stage('Approval QA') {
            steps {
                input message: "Aprobar despliegue en QA?"
            }
        }

        stage('[CD][qa] Configurar ACR credentials para Container App') {
            steps {
                script {
                    env.ENV = "qa"
                    env.API_PROVIDER_URL = "http://qa.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
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
                '''
            }
        }

        stage('[CD][qa] Deploy a Azure Container App') {
            steps {
                script {
                    env.ENV = "qa"
                    env.API_PROVIDER_URL = "http://qa.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('[CD][qa] Imprimir endpoint del Container App') {
            steps {
                script {
                    env.ENV = "qa"
                    env.API_PROVIDER_URL = "http://qa.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
                sh '''
                  ENDPOINT=$(az containerapp show \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --query properties.configuration.ingress.fqdn -o tsv)
                  echo "Endpoint del Container App ($ENV): https://$ENDPOINT"
                '''
            }
        }
        
        stage('Approval PRD') {
            steps {
                input message: "Aprobar despliegue en PRD?"
            }
        }

        stage('[CD][prd] Configurar ACR credentials para Container App') {
            steps {
                script {
                    env.ENV = "prd"
                    env.API_PROVIDER_URL = "http://prd.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
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
                '''
            }
        }

        stage('[CD][prd] Deploy a Azure Container App') {
            steps {
                script {
                    env.ENV = "prd"
                    env.API_PROVIDER_URL = "http://prd.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
                sh '''
                  echo ">>> Desplegando en $ENV"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=$ENV API_PROVIDER_URL=$API_PROVIDER_URL
                '''
            }
        }

        stage('[CD][prd] Imprimir endpoint del Container App') {
            steps {
                script {
                    env.ENV = "prd"
                    env.API_PROVIDER_URL = "http://prd.api.com"
                    env.APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
                }
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
