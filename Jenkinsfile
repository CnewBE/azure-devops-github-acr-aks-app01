pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID='06712542-8036-422e-9235-c8b008bcb707'
        AZURE_TENANT_ID='df9bad6f-9a31-4eec-b9fa-f6955eae81bd'
        CONTAINER_REGISTRY='yuchang.azurecr.io'
        RESOURCE_GROUP='SkyVe'
        REPO="yuchang"
        IMAGE_NAME="jenkinsplz"
        TAG="tag"
    }

    stages {
        stage('Example') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'myAzureCredential', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                            sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
                            sh 'az account set -s $AZURE_SUBSCRIPTION_ID'
                            sh 'az acr login --name $CONTAINER_REGISTRY --resource-group $RESOURCE_GROUP'
                            sh 'az acr build --image $REPO/$IMAGE_NAME:$TAG --registry $CONTAINER_REGISTRY --file Dockerfile . '
                        }
            }
        }
    }
}
