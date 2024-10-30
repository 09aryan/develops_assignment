pipeline {
    agent {
        docker {
            image 'node:14' // Use Node.js Docker image to ensure Node and npm are available
            args '-u root'  // Run as root user (optional, adjust if permissions are fine without this)
        }
    }
    environment {
        NODE_ENV = 'development'
        APP_PORT = '3000' // Default port, can be customized if needed
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm // Checkout code from source control (SCM) defined in the Jenkins job
            }
        }
        stage('Verify Node.js and npm Installation') {
            steps {
                echo 'Verifying Node and npm are accessible in Jenkins...'
                sh 'node -v || echo "Node.js not found"' // Checks Node.js version
                sh 'npm -v || echo "npm not found"'       // Checks npm version
            }
        }
        stage('Install dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install' // Install project dependencies from package.json
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test' // Run project tests; assumes `npm test` is defined in package.json
            }
        }
        stage('Build') {
            steps {
                echo "Building application for environment: ${env.NODE_ENV}"
                sh 'npm run build' // Adjust this if your project uses a different build command
            }
        }
        stage('Package for Deployment') {
            steps {
                echo "Packaging application..."
                sh 'tar -czf app.tar.gz *' // Compress the application files for deployment (optional)
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}" // Unique tag for Docker image
                    echo "Building Docker image with tag: ${imageTag}"
                    docker.build("my-node-app:${imageTag}") // Builds Docker image with unique tag
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'DOCKER_PASSWORD')]) {
                    script {
                        def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}" // Use the same unique tag for pushing
                        echo 'Logging in to Docker Hub...'
                        sh 'echo $DOCKER_PASSWORD | docker login -u my-dockerhub-username --password-stdin'
                        echo "Tagging Docker image as my-dockerhub-username/my-node-app:${imageTag}"
                        sh "docker tag my-node-app:${imageTag} my-dockerhub-username/my-node-app:${imageTag}"
                        echo 'Pushing Docker image to Docker Hub...'
                        sh "docker push my-dockerhub-username/my-node-app:${imageTag}"
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
            cleanWs() // Clean workspace after the build completes, regardless of success or failure
        }
    }
}
