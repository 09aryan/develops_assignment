pipeline {
    agent any
    environment {
        PATH = "${env.PATH}:/usr/local/bin"  // Ensuring Docker is accessible
        IMAGE_NAME = 'devops-21bds005--assignment'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/09aryan/develops_assignment.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t devops-21bds005--assignment .'
                }
            }
        }
        
        stage('Run Docker Container') {
            steps {
                script {
                    // Run the Docker container and map ports
                    sh 'docker run -d --name devops-21bds005--assignment -p 3000:3000 devops-21bds005--assignment'
                    sleep(10) // Allow time for the container to start up
                }
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Test the application by checking for a 200 OK response on localhost:3000
                    sh '''
                    if curl -I http://localhost:3000 | grep "200 OK"; then
                        echo "Application is running successfully"
                    else
                        echo "Application not responding"
                        exit 1
                    fi
                    '''
                }
            }
        }
    }
    post {
        always {
            // Cleanup Docker container after pipeline completes
            script {
                sh 'docker stop devops-21bds005--assignment || true'
                sh 'docker rm devops-21bds005--assignment || true'
            }
        }
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
