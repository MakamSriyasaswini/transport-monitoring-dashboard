pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "makamsriya/transport-dashboard:latest"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MakamSriyasaswini/transport-monitoring-dashboard.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        set KUBECONFIG=%KUBECONFIG%
                        kubectl apply -f deployment.yaml --validate=false
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        set KUBECONFIG=%KUBECONFIG%
                        kubectl rollout status deployment/transport-dashboard
                        kubectl get pods
                        kubectl get services
                    '''
                }
            }
        }
    }
}