# =========================
# Stage 1: Build the frontend app
# =========================
FROM node:18 AS builder

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy source code and public files
COPY src ./src
COPY public ./public

# Build the application
RUN npm run build

# =========================
# Stage 2: Serve the app using Nginx
# =========================
FROM nginx:alpine

# Copy the build artifacts from the builder stage
COPY --from=builder /app/build /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]

# =========================
# NOTE: This Dockerfile is designed to build the frontend image.
# Jenkins pipeline will handle tagging and pushing this image to Docker Hub.
# Example Jenkins pipeline steps:
#   1. docker build -t anandh15/frontend-app:latest .
#   2. docker login -u $DOCKER_USER -p $DOCKER_PASS
#   3. docker push anandh15/frontend-app:latest
# =========================
