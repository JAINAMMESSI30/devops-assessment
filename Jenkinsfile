pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'devops_assessment'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/JAINAMMESSI30/devops-assessment.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                bat 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker compose down'
                bat 'docker compose up -d --build'
            }
        }

        stage('Verify') {
            steps {
                bat 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Application is up to date.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
