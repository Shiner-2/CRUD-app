pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: regcred
"""
    }
  }

  environment {
    IMAGE_TAG = "v1.0.0" // hoặc auto: `sh(script: 'git describe --tags', returnStdout: true).trim()`
    BACKEND_IMAGE = "docker.io/shiner2/backend-api"
    FRONTEND_IMAGE = "docker.io/shiner2/frontend"
  }

  stages {
    stage('Build Backend with Kaniko') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --dockerfile=backend-api/Dockerfile \
              --context=${WORKSPACE} \
              --destination=${BACKEND_IMAGE}:${IMAGE_TAG}
          '''
        }
      }
    }

    stage('Build Frontend with Kaniko') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --dockerfile=frontend/Dockerfile \
              --context=${WORKSPACE} \
              --destination=${FRONTEND_IMAGE}:${IMAGE_TAG}
          '''
        }
      }
    }

    stage('Update Image Tag in Helm values.yaml') {
      steps {
        sh """
        sed -i 's|tag:.*|tag: "${IMAGE_TAG}"|' charts/backend-api/values.yaml
        sed -i 's|tag:.*|tag: "${IMAGE_TAG}"|' charts/frontend/values.yaml
        git config user.name "jenkins"
        git config user.email "jenkins@example.com"
        git add charts/*/values.yaml
        git commit -m "Update image tag to ${IMAGE_TAG}"
        git push origin HEAD:master
        """
      }
    }
  }
}
