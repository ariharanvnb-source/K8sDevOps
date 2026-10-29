
pipeline {
    agent any

    environment {
        IMAGE_NAME = '24mis0081/webapp'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    powershell '''
                        $env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    '''
                    bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl create deployment webapp-deployment --image=%IMAGE_NAME%:%BUILD_NUMBER% --replicas=2 --dry-run=client -o yaml | kubectl apply -f -'
                bat 'kubectl apply -f k8s\\service.yaml'
                bat 'kubectl rollout status deployment/webapp-deployment --timeout=120s'
            }
        }
    }
}
