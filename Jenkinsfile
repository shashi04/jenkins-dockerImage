pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shashivar04/python-app"
        TARGET_EC2_USER = "ubuntu"
        TARGET_EC2_HOST = "172.31.3.12"
        SSH_KEY_PATH = "~/.ssh/id_rsa.pub"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url:'https://github.com/shashi04/jenkins-dockerImage'  // Change to your actual repo
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

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-credentials']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY_PATH $TARGET_EC2_USER@$TARGET_EC2_HOST << 'EOF'
                    docker pull $DOCKER_IMAGE
                    docker stop my_container || true
                    docker rm my_container || true
                    docker run -d --name my_container -p 80:5000 $DOCKER_IMAGE
                    EOF
                    '''
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

        post {
        success {
            emailext (
                to: 'shashivardhan04@gmail.com',
                subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Good job! The build ${env.JOB_NAME} #${env.BUILD_NUMBER} was successful."
            )
        }
        failure {
            emailext (
                to: 'shashivardhan04@gmail.com',
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "The build ${env.JOB_NAME} #${env.BUILD_NUMBER} has failed. Please check Jenkins logs."
            )
        }
      }
    }
}

