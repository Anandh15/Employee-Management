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
        // Frontend Build
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
        // Backend Build
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

                // Also tag as 'latest'
                sh "docker tag ${FRONTEND_REPO}:${IMAGE_TAG} ${FRONTEND_REPO}:latest"
                sh "docker tag ${BACKEND_REPO}:${IMAGE_TAG} ${BACKEND_REPO}:latest"
                sh "docker push ${FRONTEND_REPO}:latest"
                sh "docker push ${BACKEND_REPO}:latest"
            }
        }

        // =========================
        // Deploy Containers
        // =========================
        stage('Deploy Containers') {
            steps {
                // Stop old containers
                sh 'docker rm -f frontend-container || true'
                sh 'docker rm -f backend-container || true'
                sh 'docker rm -f mysql-container || true'
                sh 'docker rm -f mongo-container || true'

                // Pull latest images
                sh "docker pull ${FRONTEND_REPO}:latest"
                sh "docker pull ${BACKEND_REPO}:latest"

                // Start MySQL
                sh '''
                docker run -d --name mysql-container \
                    -e MYSQL_ROOT_PASSWORD=password \
                    -e MYSQL_DATABASE=employee_management \
                    -p 3306:3306 \
                    mysql:8.0
                '''

                // Start MongoDB
                sh '''
                docker run -d --name mongo-container \
                    -p 27017:27017 \
                    mongo:6.0
                '''

                // Start Backend
                sh '''
                docker run -d --name backend-container \
                    -p 5000:5000 \
                    --link mysql-container:mysql \
                    --link mongo-container:mongodb \
                    ${BACKEND_REPO}:latest
                '''

                // Start Frontend
                sh "docker run -d --name frontend-container -p 3000:80 ${FRONTEND_REPO}:latest"
            }
        }
    }
}
