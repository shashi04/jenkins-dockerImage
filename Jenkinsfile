pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shashivar04/python-app:latest"
        TARGET_EC2_USER = "ubuntu"
        TARGET_EC2_HOST = "172.31.3.12"
        SSH_KEY_PATH = "$HOME/.ssh/id_rsa"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shashi04/jenkins-dockerImage'
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
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    """
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

        stage('Deploy to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-credentials', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY $TARGET_EC2_USER@$TARGET_EC2_HOST << EOF
                        docker pull $DOCKER_IMAGE
                        docker rm -f my_container
                        docker run -d --name my_container -p 80:5000 $DOCKER_IMAGE
                        EOF
                        """
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh "docker rmi $DOCKER_IMAGE || true"
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker logout"
            }
        }

        success {
            emailext (
                to: 'shashivardhan04@gmail.com',
                subject: "✅ Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Good job! The build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> was successful.</p>
                         <p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                mimeType: 'text/html'
            )
        }

        failure {
            emailext (
                to: 'shashivardhan04@gmail.com',
                subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>The build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> has failed.</p>
                         <p>Please check Jenkins logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                mimeType: 'text/html'
            )
        }
    }
}
