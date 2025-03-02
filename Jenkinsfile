pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shashivar04/python-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-repo.git'  // Change to your actual repo
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh "docker push $DOCKER_IMAGE"
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh "docker rmi $DOCKER_IMAGE"
                }
            }
        }
    }
}
