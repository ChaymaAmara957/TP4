pipeline {
    agent any

    environment {
        // Dossiers possibles de docker.exe (Docker Desktop "tous les utilisateurs" ou "mon utilisateur")
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build images') {
            steps {
                bat 'docker build -t mern-server ./Server'
                bat 'docker build -t mern-client ./Client'
            }
        }

        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                 usernameVariable: 'DOCKER_USER',
                                 passwordVariable: 'DOCKER_PASS')]) {
                    bat 'echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin'
                    bat 'docker tag mern-server %DOCKER_USER%/mern-server:latest'
                    bat 'docker tag mern-client %DOCKER_USER%/mern-client:latest'
                    bat 'docker push %DOCKER_USER%/mern-server:latest'
                    bat 'docker push %DOCKER_USER%/mern-client:latest'
                }
            }
        }

        stage('Deploy (compose)') {
            steps {
                bat '''
                    docker compose version >nul 2>&1
                    if %errorlevel%==0 (
                        docker compose down
                        docker compose up -d --build
                    ) else (
                        docker-compose down
                        docker-compose up -d --build
                    )
                '''
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
    }
}
