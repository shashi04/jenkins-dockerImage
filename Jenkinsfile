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
            emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
        }
    }

    // post {
    //     always {
    //         script {
    //             def emailSubject
    //             def emailBody
    //             def recipientEmail

    //             if (currentBuild.result == "SUCCESS") {
    //                 emailSubject = "Pipeline Success: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}"
    //                 emailBody = "The pipeline run for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} was successful. You can view the details at ${env.BUILD_URL}"
    //                 recipientEmail = "shashivardhan04@gmail.com"
    //             }
    //             else {
    //                 emailSubject = "Pipeline Failure: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}"
    //                 emailBody = "The pipeline run for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} has failed. You can view the details at ${env.BUILD_URL}"
    //                 recipientEmail = "shashivardhan04@gmail.com"
    //             }

    //             emailext (
    //                 subject: emailSubject,
    //                 body: emailBody,
    //                 to: recipientEmail
    //             )
    //         }
    //     }
    // }
}
