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
                        sh 'docker build -t hello-world:v2 .'
                        sh 'docker tag hello-world:v2 yuchang.azurecr.io/hello-world:v2'
                        sh 'docker push yuchang.azurecr.io/hello-world:v2'
            }
        }
    }
}
