pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'anandh15/frontend-app'
        BACKEND_REPO  = 'anandh15/backend-app'
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}"  // short commit hash
    }

    stages {
        // =========================
        // Checkout Code
        // =========================
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Anandh15/Employee-Management.git'
            }
        }

        // =========================
        // Build Frontend Image
        // =========================
        stage('Frontend - Build Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        // =========================
        // Build Backend Image
        // =========================
        stage('Backend - Build Docker Image') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        // =========================
        // Docker Hub Login
        // =========================
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        // =========================
        // Push Images to Docker Hub
        // =========================
        stage('Push Docker Images') {
            steps {
                sh """
                docker push ${FRONTEND_REPO}:${IMAGE_TAG}
                docker push ${BACKEND_REPO}:${IMAGE_TAG}

                docker tag ${FRONTEND_REPO}:${IMAGE_TAG} ${FRONTEND_REPO}:latest
                docker tag ${BACKEND_REPO}:${IMAGE_TAG} ${BACKEND_REPO}:latest

                docker push ${FRONTEND_REPO}:latest
                docker push ${BACKEND_REPO}:latest
                """
            }
        }

        // =========================
        // Deploy with Docker Compose
        // =========================
        stage('Deploy Containers') {
            steps {
                sh '''
                # Stop existing containers
                docker compose down || true

                # Pull latest images
                docker pull ${FRONTEND_REPO}:latest
                docker pull ${BACKEND_REPO}:latest

                # Start everything with docker-compose.yml
                docker compose up -d
                '''
            }
        }
    }
}
