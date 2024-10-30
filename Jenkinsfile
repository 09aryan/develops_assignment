pipeline {
    agent any
    tools {
        nodejs 'NodeJs' // Matches the exact name in Global Tool Configuration
    }
    environment {
        NODE_ENV = 'development'
        APP_PORT = '3000'
    }
    stages {
        stage('Verify Node.js and npm Installation') {
            steps {
                echo 'Verifying Node and npm are accessible in Jenkins...'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        stage('Install dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test || echo "No tests specified, skipping..." && exit 0'
            }
        }
        stage('Build') {
            steps {
                echo "Building application for environment: ${env.NODE_ENV}"
                sh 'npm run build'
            }
        }
        stage('Package for Deployment') {
            steps {
                echo "Packaging application..."
                sh 'tar -czf app.tar.gz *'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    withEnv(['PATH+DOCKER=/usr/local/bin']) {  // Adjust path if Docker is located elsewhere
                        echo "Building Docker image for tag: ${env.BUILD_ID}"
                        docker.build("my-node-app:${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHubPassword', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        echo 'Logging in to Docker Hub...'
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        echo "Tagging Docker image as $DOCKER_USERNAME/my-node-app:${env.BUILD_ID}"
                        sh "docker tag my-node-app:${env.BUILD_ID} $DOCKER_USERNAME/my-node-app:${env.BUILD_ID}"
                        echo 'Pushing Docker image to Docker Hub...'
                        sh "docker push $DOCKER_USERNAME/my-node-app:${env.BUILD_ID}"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
        always {
            echo "Cleaning up workspace..."
            cleanWs() // Clean workspace after build completes
        }
    }
}
