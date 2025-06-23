pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        GITHUB_CREDENTIALS = credentials('github')
        FRONTEND_IMAGE = 'shiner2/frontend'
        BACKEND_IMAGE = 'shiner2/backend-api'
        GIT_TAG = ''
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
                script {
                    GIT_TAG = sh(returnStdout: true, script: "git describe --tags --abbrev=0").trim()
                    echo "Building with Git tag: ${GIT_TAG}"
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh "docker build -t ${FRONTEND_IMAGE}:${GIT_TAG} frontend"
            }
        }

        stage('Build Backend Image') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE}:${GIT_TAG} backend-api"
            }
        }

        stage('Login to Docker Hub & Push Images') {
            steps {
                sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                sh "docker push ${FRONTEND_IMAGE}:${GIT_TAG}"
                sh "docker push ${BACKEND_IMAGE}:${GIT_TAG}"
                sh "docker logout"
            }
        }

        stage('Update config repo values.yaml') {
            steps {
                sh 'rm -rf config-repo'
                sh 'git config --global user.name "jenkins"'
                sh 'git config --global user.email "jenkins@example.com"'
                sh "git clone https://${GITHUB_CREDENTIALS_USR}:${GITHUB_CREDENTIALS_PSW}@github.com/Shiner-2/CRUD-config.git config-repo"

                // Update values.yaml tags
                sh """
                sed -i 's|\\(backend-api:\\s*\\n\\s*replicaCount:.*\\n\\s*image:\\s*\\n\\s*repository: shiner2/backend-api\\n\\s*tag: \\).*|\\1${GIT_TAG}|' config-repo/values.yaml
                sed -i 's|\\(frontend:\\s*\\n\\s*replicaCount:.*\\n\\s*image:\\s*\\n\\s*repository: shiner2/frontend\\n\\s*tag: \\).*|\\1${GIT_TAG}|' config-repo/values.yaml
                """

                dir('config-repo') {
                    sh '''
                    git add values.yaml
                    git commit -m "Update image tags to ${GIT_TAG}"
                    git push origin main
                    '''
                }
            }
        }
    }
}
