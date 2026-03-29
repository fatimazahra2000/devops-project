pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "fatimazahraab/todo-app:v1"
    }

    stages {

        // Checkout du repo
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        //  Login DockerHub
        stage('Login to DockerHub') {
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

        // Installer les dépendances Python
        stage('Install dependencies') {
            steps {
                bat 'py -m pip install --upgrade pip'
                bat 'py -m pip install -r requirements.txt'
                bat 'py -m pip install pytest'
            }
        }

        // Lancer les tests
        stage('Test') {
            steps {
                bat 'py -m pytest -v'
            }
        }

        //  Build Docker Image
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        //  Push Docker Image
        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%'
            }
        }

        //  Déploiement Kubernetes sécurisé avec kubeconfig
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/deployment.yaml
                    kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/service.yaml
                    '''
                }
            }
        }

    } // end stages

    post {
        success {
            echo "Pipeline SUCCESS"
        }
        failure {
            echo "Pipeline FAILED - check logs"
        }
    }
}