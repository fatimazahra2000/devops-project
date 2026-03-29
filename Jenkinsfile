pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "fatimazahraab/todo-app"
        DOCKER_TAG   = "${env.GIT_COMMIT.take(7)}" // Tag unique par commit
        IMAGE        = "${DOCKER_IMAGE}:${DOCKER_TAG}"
    }

    // Déclenche le pipeline automatiquement via GitHub webhook
    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Login') {
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
                echo "🛠 Build Docker image : ${IMAGE}"
                bat 'docker build -t %IMAGE% .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Push Docker image : ${IMAGE}"
                bat 'docker push %IMAGE%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    echo "Deploying image ${IMAGE} to Kubernetes"
                    bat """
                    kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/deployment.yaml
                    kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/service.yaml
                    kubectl --kubeconfig=%KUBECONFIG% set image deployment/todo-app todo-app=%IMAGE% --record
                    kubectl --kubeconfig=%KUBECONFIG% rollout status deployment/todo-app
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS - Image déployée : ${IMAGE}"
        }
        failure {
            echo "Pipeline FAILED - Vérifier les logs"
        }
    }
}