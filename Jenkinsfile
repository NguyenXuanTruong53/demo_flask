pipeline {
    agent any
    
    stages {
        stage('Build and Run Docker') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t my-docker-image .'
                    
                    // Run Docker container
                    sh 'docker run -d --name my-docker-container my-docker-image'
                }
            }
        }
    }
    
    post {
        always {
            // Clean up - stop and remove the container
            sh 'docker stop my-docker-container'
            sh 'docker rm my-docker-container'
        }
    }
}
