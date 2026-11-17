
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE = 'hibatallahnakaa/monapp'
        TAG = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Docker') {
            steps {
                bat 'whoami'
                bat 'docker version'
                bat 'docker run --rm nginx:alpine nginx -v'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker version'
                bat 'docker build -t %IMAGE%:%TAG% .'
            }
        }

        stage('Smoke Test') {
            steps {
                bat '''
                docker rm -f monapp_test 2>nul || ver > nul
                docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
                ping -n 3 127.0.0.1 > nul
                curl -I http://localhost:8081 | find "200 OK"
                docker rm -f monapp_test
                '''
            }
        }

        stage('Push (Docker Hub)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    bat '''
                    powershell -NoProfile -NonInteractive -Command "$env:DOCKERHUB_TOKEN | docker login -u $env:DOCKERHUB_USER --password-stdin; if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }"
                    if errorlevel 1 exit /b 1

                    docker tag %IMAGE%:%TAG% %IMAGE%:latest
                    if errorlevel 1 exit /b 1

                    docker push %IMAGE%:%TAG%
                    if errorlevel 1 exit /b 1

                    docker push %IMAGE%:latest
                    if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build+Test+Push OK'
        }
        failure {
            echo 'Build/Tests/Push KO'
        }
    }
}
