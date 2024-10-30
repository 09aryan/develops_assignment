pipeline {
    agent any
    environment {
        NODE_ENV = 'development'
        PORT = '3000'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("my-node-app:${env.BUILD_ID}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u my-dockerhub-username --password-stdin'
                    sh "docker tag my-node-app:${env.BUILD_ID} my-dockerhub-username/my-node-app:${env.BUILD_ID}"
                    sh "docker push my-dockerhub-username/my-node-app:${env.BUILD_ID}"
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'API_KEY', variable: 'API_KEY'), string(credentialsId: 'DATABASE_URL', variable: 'DATABASE_URL')]) {
                    sh 'echo "Deploying with API_KEY and DATABASE_URL"'
                    // Additional deployment steps can go here
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
    }
}
