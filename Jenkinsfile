pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "yuvrajahire/yuvraj-node-app"
        DOCKER_CREDENTIALS = 'dockerhub-credentials' // Jenkins credentials ID
        APP_NAME = "yuvraj-node-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    triggers {
        githubPush() // Auto trigger when code is pushed
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yuvrajahire/Auto-Provisioned-CI-CD-infrastructure.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} -f ./docker/Dockerfile ./docker
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Blue-Green Deploy') {
            steps {
                script {
                    sh "chmod +x shell-scripts/deploy.sh"
                    sh "./shell-scripts/deploy.sh ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}

