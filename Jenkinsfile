pipeline {
    agent any

    environment {
        // Defining variables to keep the code clean and reusable
        REGISTRY_USER = 'akshaykadam45'
        REPO_NAME     = 'myrepo'
        IMAGE_NAME    = 'myimage'
        // Jenkins automatically provides ${BUILD_NUMBER}
        FULL_IMAGE_TAG = "${REGISTRY_USER}/${REPO_NAME}:${IMAGE_NAME}${BUILD_NUMBER}"
        
        // Pulling the secret password securely from Jenkins Credentials
        DOCKER_PASSWORD = credentials('docker-hub-password')
        DOCKER_USER     = 'akshaykadam-45' // Replace with your actual Docker username variable if needed
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls the latest code from your repository
                checkout scm
            }
        }

        stage('Docker Build & Tag') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_NAME} ."
                
                echo "Tagging Docker image..."
                sh "docker tag ${IMAGE_NAME} ${FULL_IMAGE_TAG}"
            }
        }

        stage('Docker Push') {
            steps {
                echo "Logging into Docker Hub..."
                // Using the environment variables safely in the shell
                sh "echo '${DOCKER_PASSWORD}' | docker login -u '${DOCKER_USER}' --password-stdin"
                
                echo "Pushing image to registry..."
                sh "docker push ${FULL_IMAGE_TAG}"
            }
        }

        stage('Update Manifest & Deploy') {
            steps {
                echo "Updating deployment manifest with new image tag..."
                // Using double quotes so Jenkins can resolve the ${FULL_IMAGE_TAG} variable inside the sed command
                sh "sed -i 's|image: httpd|image: ${FULL_IMAGE_TAG}|' mydep.yaml"
                
                echo "Applying manifests to Kubernetes cluster..."
                sh "kubectl apply -f ."
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images to save space..."
            sh "docker rmi ${IMAGE_NAME} ${FULL_IMAGE_TAG} || true"
            // Securely logging out of Docker
            sh "docker logout || true"
        }
    }
}
