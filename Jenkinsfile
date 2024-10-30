pipeline {
    agent any
    environment {
        NODE_ENV = 'development'
        APP_PORT = '3000' // Default port, can be customized
    }
    stages {
        stage('Verify Node.js and npm Installation') {
            steps {
                echo 'Verifying Node and npm are accessible in Jenkins...'
                sh 'which node || echo "Node.js not found"'
                sh 'which npm || echo "npm not found"'
                sh 'node -v || echo "Node.js version not found"'
                sh 'npm -v || echo "npm version not found"'
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
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                echo "Building application for environment: ${env.NODE_ENV}"
                sh 'npm run build' // Adjust if your project uses a specific build command
            }
        }
        stage('Package for Deployment') {
            steps {
                echo "Packaging application..."
                sh 'tar -czf app.tar.gz *' // Compresses the app files for deployment
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image for tag: ${env.BUILD_ID}"
                    docker.build("my-node-app:${env.BUILD_ID}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'DOCKER_PASSWORD')]) {
                    script {
                        echo 'Logging in to Docker Hub...'
                        sh 'echo $DOCKER_PASSWORD | docker login -u my-dockerhub-username --password-stdin'
                        echo "Tagging Docker image as my-dockerhub-username/my-node-app:${env.BUILD_ID}"
                        sh "docker tag my-node-app:${env.BUILD_ID} my-dockerhub-username/my-node-app:${env.BUILD_ID}"
                        echo 'Pushing Docker image to Docker Hub...'
                        sh "docker push my-dockerhub-username/my-node-app:${env.BUILD_ID}"
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
