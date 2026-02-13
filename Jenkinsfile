pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aisalkyn85/manual-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_DEPLOYMENT = "manual-app"
        KUBE_CONTAINER = "manual-app"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing npm dependencies..."
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing image to DockerHub..."
                sh """
                docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Updating Kubernetes deployment..."
                sh """
                kubectl set image deployment/${KUBE_DEPLOYMENT} \
                ${KUBE_CONTAINER}=${DOCKER_IMAGE}:${IMAGE_TAG} --record
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                echo "Checking rollout status..."
                sh """
                kubectl rollout status deployment/${KUBE_DEPLOYMENT}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            echo "Pipeline finished."
        }
    }
}
