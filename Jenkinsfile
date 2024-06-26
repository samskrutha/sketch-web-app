pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'sketch-git'
        REGISTRY_URL = 'samskrutha'
        REGISTRY_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_TAG = "latest-${BUILD_NUMBER}"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${REGISTRY_URL}/sketch-web-app:${IMAGE_TAG} .'
                    sh 'docker images'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh 'docker push ${REGISTRY_URL}/sketch-web-app:${IMAGE_TAG}'
                    }
                }
            }
        }
        stage('Notify GitOps Repo') {
            steps {
                withCredentials([string(credentialsId: "${GIT_CREDENTIALS_ID}", variable: 'GIT_TOKEN')]) {
                    script {
                        git branch: 'main', url: "https://${GIT_TOKEN}@github.com/samskrutha/sketch-argocd.git"
                        sh 'git branch --set-upstream-to=origin/main main'
                        sh 'echo "Update manifest to use new image"'
                        sh 'git pull origin main'
                        sh 'echo "Before sed:"'
                        sh 'cat apps/sketch-web-app/deployment.yaml'
                        sh 'sed -i "s|image: .*$|image: ${REGISTRY_URL}/sketch-web-app:${IMAGE_TAG}|" apps/sketch-web-app/deployment.yaml'
                        sh 'echo "After sed:"'
                        sh 'cat apps/sketch-web-app/deployment.yaml'
                        
                        script {
                            def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                            if (changes) {
                                sh 'git add apps/sketch-web-app/deployment.yaml'
                                sh 'git commit -m "Update image to ${REGISTRY_URL}/sketch-web-app:${IMAGE_TAG}"'
                                sh 'git push origin main'
                            } else {
                                echo 'No changes to commit'
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Build, push, and GitOps update successful.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
