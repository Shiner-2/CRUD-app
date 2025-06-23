pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "shiner2/backend-api"
    FRONTEND_IMAGE = "shiner2/frontend"
    IMAGE_TAG = "${GIT_TAG_NAME ?: 'latest'}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Push Backend') {
      steps {
        dir('backend-api') {
          script {
            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
              docker.build("${BACKEND_IMAGE}:${IMAGE_TAG}").push()
            }
          }
        }
      }
    }

    stage('Build and Push Frontend') {
      steps {
        dir('frontend') {
          script {
            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
              docker.build("${FRONTEND_IMAGE}:${IMAGE_TAG}").push()
            }
          }
        }
      }
    }

    stage('Update Helm Config') {
      steps {
        echo "Cập nhật Helm values.yaml (nếu cần)"
      }
    }
  }
}
