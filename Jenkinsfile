pipeline {
  agent any
  environment {
    IMAGE_REPO = '${IMAGE_REPO}' //ACR 주소
    IMAGE_NAME = '${IMAGE_NAME}' //Image 이름
    IMAGE_TAG = "${env.BUILD_NUMBER}" //수정 필요 x(Jenkins 현재 빌드 Number 환경변수)
    ENVIRONMENT = '${ENVIRONMENT}' //환경
    HELM_VALUES = '${HELM_VALUES}' //Helm Values(Image_tag 변경)
  }
  stages {
    stage('Build') {
        steps {
            sh './mvnw compile'
        }
    }
    stage('Unit Test') {
        steps {
            sh './mvnw test'
        }
        post {
            always {
                junit 'target/surefire-reports/*.xml'
                step([ $class: 'JacocoPublisher' ])
            }
        }
    }
    stage('Static Code Analysis') {
        steps {
            configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                sh './mvnw sonar:sonar -s $MAVEN_SETTINGS'
            }
        }
    }
    stage('Package') {
        steps {
            sh './mvnw package -DskipTests'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
    stage('Build Docker image') {
        steps {
            echo 'The build number is ${IMAGE_TAG}'
            sh 'docker build --build-arg ENVIRONMENT=${ENVIRONMENT} -t ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .'
        }
    }
    stage('Push Docker image') {
        steps {
            withCredentials([azureServicePrincipal('azure_service_principal')]) {
                echo '---------az login------------'
                sh '''
                az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                az account set -s $AZURE_SUBSCRIPTION_ID
                '''
                sh 'az acr login --name mungtaregistry'
                sh 'docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'az logout'
            }
        }
    }
    stage('Clean Docker image') {
        steps {
            echo '---------Clean image------------'
            sh 'docker rmi ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}'
        }
    }
    stage('Update manifest') {
        steps {
          sh """
            git config --global user.name "${GITHUB_NAME}"
            git config --global user.email "${GITHUB_EMAIL}"
            git config --global credential.helper cache
            git config --global push.default simple
          """

          git url: "${HELM_CHART}", credentialsId: 'mungta_github_ssh', branch: 'main'
          sh """
            sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/g' ${HELM_VALUES}
            git add ${HELM_VALUES}
            git commit -m 'Update Docker image tag: ${IMAGE_TAG}'
          """

          sshagent (credentials: ['mungta_github_ssh']) {
            sh 'git push origin main'
          }
        }
    }
}
