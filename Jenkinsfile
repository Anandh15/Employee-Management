pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'anandh15/frontend-app'
        BACKEND_REPO = 'anandh15/backend-app'
        IMAGE_TAG = "${env.GIT_COMMIT.take(7)}" // unique tag using commit hash
    }

    stages {
        // =========================
        // Checkout Code
        // =========================
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Anandh15/Employee-Management.git'
            }
        }

        // =========================
        // Frontend Build & Docker
        // =========================
        stage('Frontend - Install Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }

        stage('Frontend - Build Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        // =========================
        // Backend Build & Docker
        // =========================
        stage('Backend - Build Docker Image') {
            steps {
                dir('backend') {
                    sh "docker build --platform linux/amd64 -t ${BACKEND_REPO}:${IMAGE_TAG} ."
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
        // Push Docker Images
        // =========================
        stage('Push Docker Images') {
            steps {
                sh "docker push ${FRONTEND_REPO}:${IMAGE_TAG}"
                sh "docker push ${BACKEND_REPO}:${IMAGE_TAG}"
            }
        }

        // =========================
        // Deploy Fullstack App with Docker Compose
        // =========================
        stage('Deploy Fullstack with DB') {
            steps {
                // Stop existing containers
                sh "docker-compose down"

                // Pull latest images for backend/frontend if needed
                // sh "docker-compose pull"

                // Start all services including MySQL, MongoDB, backend, and frontend
                sh "docker-compose up -d --build"

                // Optional: Wait for databases to be healthy
                sh "docker-compose ps"
            }
        }
    }
}
