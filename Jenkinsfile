pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOY_TO_K8S',
            choices: ['YES', 'NO'],
            description: 'Choose whether to deploy to Kubernetes'
        )
    }

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

        stage('Test Application') {
            parallel {

                stage('Test HTML') {
                    steps {
                        bat '''
                            if not exist index.html exit /b 1
                            findstr /I /C:"Transport" index.html
                            if errorlevel 1 exit /b 1
                        '''
                    }
                }

                stage('Check Deployment Files') {
                    steps {
                        bat '''
                            if not exist Dockerfile exit /b 1
                            if not exist deployment.yaml exit /b 1
                            docker --version
                            kubectl version --client
                        '''
                    }
                }
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
                        credentialsId: 'dockerhub-fresh',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        powershell -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin; if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }"
                        if errorlevel 1 exit /b 1
                        docker push %DOCKER_IMAGE%
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression {
                    params.DEPLOY_TO_K8S == 'YES'
                }
            }

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
                        kubectl rollout status deployment/transport-dashboard
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            when {
                expression {
                    params.DEPLOY_TO_K8S == 'YES'
                }
            }

            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        set KUBECONFIG=%KUBECONFIG%
                        kubectl get deployments
                        kubectl get pods
                        kubectl get services
                    '''
                }
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: 'index.html,Dockerfile,deployment.yaml',
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
