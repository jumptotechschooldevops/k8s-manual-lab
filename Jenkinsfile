pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aisalkyn85/manual-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CD_REPO = "https://github.com/jumptotechschooldevops/manual-app-helm.git"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning CI repository..."
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

        stage('Update CD Repo') {
            steps {
                echo "Updating Helm values.yaml in CD repository..."

                sh """
                rm -rf cd-repo
                git clone ${CD_REPO} cd-repo
                cd cd-repo

                # Update image tag in values.yaml
                sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/' values.yaml

                git config user.email "jenkins@jumptotech.com"
                git config user.name "jenkins"

                git add values.yaml
                git commit -m "Update image tag to ${IMAGE_TAG}"
                git push
                """
            }
        }
    }

    post {
        success {
            echo "CI Pipeline completed successfully!"
        }
        failure {
            echo "CI Pipeline failed!"
        }
        always {
            echo "Pipeline finished."
        }
    }
}

