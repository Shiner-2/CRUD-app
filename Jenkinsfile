pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "shiner2/backend-api"
    FRONTEND_IMAGE = "shiner2/frontend"
    IMAGE_TAG = "${GIT_TAG_NAME}"
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
            docker.build("${BACKEND_IMAGE}:${IMAGE_TAG}").push()
          }
        }
      }
    }

    stage('Build and Push Frontend') {
      steps {
        dir('frontend') {
          script {
            docker.build("${FRONTEND_IMAGE}:${IMAGE_TAG}").push()
          }
        }
      }
    }

    stage('Update Helm Config (optional)') {
      steps {
        echo "Update image tag trong config repo (bạn có thể làm thủ công nếu chưa có config repo riêng)"
      }
    }
  }

  triggers {
    // Nếu dùng plugin hoặc webhook để phát hiện tag push
  }
}
