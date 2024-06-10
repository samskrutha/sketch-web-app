pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'github-token'
        REGISTRY_URL = 'samskrutha'
        REGISTRY_CREDENTIALS = 'dockerhub-credentials'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${REGISTRY_URL}/sketch-web-app:latest .'
                    sh 'docker images'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh 'docker push ${REGISTRY_URL}/sketch-web-app:latest'
                    }
                }
            }
        }
        stage('Notify GitOps Repo') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/samskrutha/sketch-argocd.git', credentialsId: "${GIT_CREDENTIALS_ID}"
                    sh 'echo "Update manifest to use new image"'
                    sh 'git pull'
                    sh 'sed -i "s|image: .*$|image: ${REGISTRY_URL}/sketch-web-app:latest|" apps/sketch-web-app/deployment.yaml'
                    sh 'git commit -am "Update image to ${REGISTRY_URL}/sketch-web-app:latest"'
                    sh 'git push'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
