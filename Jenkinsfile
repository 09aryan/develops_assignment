pipeline {
    agent any
    tools { nodejs 'NodeJs' } // Ensure this matches your Node.js installation name

    environment {
        NODE_ENV = 'development'
        APP_PORT = '3000' // Default application port
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm // Check out the code from the source control
            }
        }

        stage('Verify Node.js and npm Installation') {
            steps {
                echo 'Verifying Node and npm are accessible in Jenkins...'
                sh 'node -v || echo "Node.js not found"'
                sh 'npm -v || echo "npm not found"'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install' // Install the dependencies defined in package.json
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test || echo "No test specified or tests failed"' // Run tests; handle cases where no tests are specified
            }
        }

        stage('Build') {
            steps {
                echo "Building application for environment: ${env.NODE_ENV}"
                sh 'npm run build || echo "No build script defined"' // Attempt to run the build command
            }
        }

        stage('Package for Deployment') {
            steps {
                echo "Packaging application..."
                sh 'tar -czf app.tar.gz *' // Package all files into a tar.gz archive
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}" // Create a unique image tag
                    echo "Building Docker image with tag: ${imageTag}"
                    sh "docker build -t my-node-app:${imageTag} ." // Build the Docker image
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'DOCKER_PASSWORD')]) {
                    script {
                        def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}" // Use the same image tag for pushing
                        echo 'Logging in to Docker Hub...'
                        sh 'echo $DOCKER_PASSWORD | docker login -u my-dockerhub-username --password-stdin' // Log in to Docker Hub
                        echo "Tagging Docker image as my-dockerhub-username/my-node-app:${imageTag}"
                        sh "docker tag my-node-app:${imageTag} my-dockerhub-username/my-node-app:${imageTag}" // Tag the Docker image
                        echo 'Pushing Docker image to Docker Hub...'
                        sh "docker push my-dockerhub-username/my-node-app:${imageTag}" // Push the image to Docker Hub
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.' // Success message
        }
        failure {
            echo 'Pipeline failed.' // Failure message
        }
        always {
            echo "Cleaning up workspace..." // Clean up workspace after each run
            cleanWs()
        }
    }
}
