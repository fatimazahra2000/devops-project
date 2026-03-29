pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "fatimazahraab/todo-app:v1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Login to DockerHub (fix pull/build)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'py -m pip install -r requirements.txt'
                bat 'py -m pip install pytest'
            }
        }

        stage('Test') {
            steps {
                bat 'py -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS "
        }
        failure {
            echo "Pipeline FAILED check logs"
        }
    }
}