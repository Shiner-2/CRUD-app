pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        GITHUB_CREDENTIALS = credentials('github')
        FRONTEND_IMAGE = 'shiner2/frontend'
        BACKEND_IMAGE = 'shiner2/backend-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check for changes') {
            steps {
                script {
                    def lastBuiltCommit = ''
                    if (fileExists('.last_build_commit')) {
                        lastBuiltCommit = readFile('.last_build_commit').trim()
                    }
                    def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    echo "Last built commit: ${lastBuiltCommit}"
                    echo "Current commit: ${currentCommit}"

                    if (lastBuiltCommit == currentCommit) {
                        echo "No new commits since last build. Skipping build."
                        currentBuild.result = 'SUCCESS'
                        error("No changes detected, aborting build.")
                    } else {
                        echo "New commit detected, proceed with build."
                    }
                }
            }
        }

        stage('Get Git Tag') {
            steps {
                script {
                    env.GIT_TAG = sh(returnStdout: true, script: "git describe --tags --abbrev=0 || echo 'latest'").trim()
                    echo "Using Git tag: ${env.GIT_TAG}"
                }
            }
        }

        stage('Build and Push') {
            steps {
                sh "docker build -t ${FRONTEND_IMAGE}:${GIT_TAG} frontend"
                sh "docker build -t ${BACKEND_IMAGE}:${GIT_TAG} backend-api"

                sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                sh "docker push ${FRONTEND_IMAGE}:${GIT_TAG}"
                sh "docker push ${BACKEND_IMAGE}:${GIT_TAG}"
                sh "docker logout"
            }
        }

        stage('Update values.yaml & Push') {
            steps {
                sh """
                sed -i 's|\\(backend-api:\\s*\\n\\s*replicaCount:.*\\n\\s*image:\\s*\\n\\s*repository: shiner2/backend-api\\n\\s*tag: \\).*|\\1${GIT_TAG}|' values.yaml
                sed -i 's|\\(frontend:\\s*\\n\\s*replicaCount:.*\\n\\s*image:\\s*\\n\\s*repository: shiner2/frontend\\n\\s*tag: \\).*|\\1${GIT_TAG}|' values.yaml
                """

                sh 'git config user.name "jenkins"'
                sh 'git config user.email "jenkins@example.com"'
                sh 'git add values.yaml'
                sh "git commit -m 'Update image tags to ${GIT_TAG}' || echo 'No changes to commit'"
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh "git push https://${GIT_USER}:${GIT_PASS}@github.com/Shiner-2/CRUD-app.git HEAD:refs/heads/main"
                }
            }
        }

        stage('Save current commit') {
            steps {
                script {
                    def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    writeFile file: '.last_build_commit', text: currentCommit
                }
            }
        }
    }
}
