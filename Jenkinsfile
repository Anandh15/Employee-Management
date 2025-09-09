pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'anandh15/frontend-app'
        BACKEND_REPO  = 'anandh15/backend-app'
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}" // unique tag using commit hash
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
        // Deploy Containers
        // =========================
        stage('Deploy Containers') {
            steps {
                sh """
                    # Stop and remove old containers if running
                    docker rm -f frontend-container || true
                    docker rm -f backend-container || true

                    # Pull latest images
                    docker pull ${FRONTEND_REPO}:${IMAGE_TAG}
                    docker pull ${BACKEND_REPO}:${IMAGE_TAG}

                    # Run frontend container on port 3000
                    docker run -d --name frontend-container -p 3000:3000 ${FRONTEND_REPO}:${IMAGE_TAG}

                    # Run backend container on port 5000 (instead of 8080, since Jenkins uses 8080)
                    docker run -d --name backend-container -p 5000:8080 ${BACKEND_REPO}:${IMAGE_TAG}
                """
            }
        }
    }
}
