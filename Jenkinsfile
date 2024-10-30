pipeline {
    agent any
   tools { nodejs 'NodeJs' } 

    environment {
        NODE_ENV = 'development'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Verify Node.js and npm Installation') {
            steps {
                echo 'Verifying Node and npm are accessible in Jenkins...'
                sh 'node -v || echo "Node.js not found"'
                sh 'npm -v || echo "npm not found"'
            }
        }

        stage('Install dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        // Commented out the Test stage for now
        /*
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        */

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
                    def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}"
                    echo "Building Docker image with tag: ${imageTag}"
                    sh "docker build -t my-node-app:${imageTag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'DOCKER_PASSWORD')]) {
                    script {
                        def imageTag = "${env.JOB_NAME}-${env.BUILD_NUMBER}"
                        echo 'Logging in to Docker Hub...'
                        sh 'echo $DOCKER_PASSWORD | docker login -u my-dockerhub-username --password-stdin'
                        echo "Tagging Docker image as my-dockerhub-username/my-node-app:${imageTag}"
                        sh "docker tag my-node-app:${imageTag} my-dockerhub-username/my-node-app:${imageTag}"
                        echo 'Pushing Docker image to Docker Hub...'
                        sh "docker push my-docker-app:${imageTag}"
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
            cleanWs()
        }
    }
}
