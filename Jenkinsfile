pipeline {
    agent { label 'master' }

    environment {
        // Define your DockerHub credentials ID here (create this in Jenkins Credentials)
        DOCKERHUB_CREDENTIALS = '2026@Ajay003'
        DOCKER_IMAGE = "ajaykumarrgn/sample-nodejs-app"
        // Use Jenkins BUILD_NUMBER as the image tag for versioning
        IMAGE_TAG = "${env.BUILD_NUMBER}" 
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker Image...'
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing image to Docker registry...'
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes Cluster...'
                // 1. Replace the placeholder in the yaml with the actual built image tag
                sh "sed -i 's|DOCKER_IMAGE_PLACEHOLDER|${DOCKER_IMAGE}:${IMAGE_TAG}|g' k8s-manifest.yaml"
                // 2. Apply to Kubernetes
                sh "cd /home/ubuntu/jenkins-k8s-project"
                sh "kubectl apply -f k8s-manifest.yaml"
            }
        }
    }

    post {
        success {
            echo "Successfully deployed version ${IMAGE_TAG} to Kubernetes!"
        }
        failure {
            echo "Pipeline Failed! Check logs."
        }
    }
}
