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
                // Stop old containers (if any)
                sh '''
                docker rm -f frontend-container || true
                docker rm -f backend-container || true
                docker rm -f mysql-container || true
                docker rm -f mongo-container || true
                '''

                // Pull latest images
                sh '''
                docker pull ${FRONTEND_REPO}:latest
                docker pull ${BACKEND_REPO}:latest
                '''

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

                // Start Backend (depends on MySQL + MongoDB)
                sh '''
                docker run -d --name backend-container \
                    -p 5000:8080 \
                    --link mysql-container:mysql \
                    --link mongo-container:mongodb \
                    -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/employee_management \
                    -e SPRING_DATASOURCE_USERNAME=root \
                    -e SPRING_DATASOURCE_PASSWORD=password \
                    -e SPRING_DATA_MONGODB_URI=mongodb://mongodb:27017/employee_management \
                    ${BACKEND_REPO}:latest
                '''

                // Start Frontend (depends on Backend)
                sh '''
                docker run -d --name frontend-container \
                    -p 3000:80 \
                    --link backend-container:backend \
                    ${FRONTEND_REPO}:latest
                '''
            }
        }
    }
}
