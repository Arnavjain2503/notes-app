pipeline {
    agent any 
    
    stages {
        stage("Clone Code") {
            steps {
                echo "Cloning the code"
                git url: "https://github.com/Arnavjain2503/notes-app.git", branch: "main"
            }
        }

        stage("Build") {
            steps {
                echo "Building the image"
                sh "docker build -t django-note-app ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "docker tag django-note-app ${env.dockerHubUser}/django-note-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker push ${env.dockerHubUser}/django-note-app:latest"
                }
            }
        }

        stage("Deploy to App EC2") {
            steps {
                echo "Deploying the container on App EC2"

                withCredentials([sshUserPrivateKey(credentialsId: "jenkins-ssh-key", keyFileVariable: "SSH_KEY")]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@<APP_EC2_PUBLIC_IP> << EOF
                        docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}
                        docker pull ${env.dockerHubUser}/django-note-app:latest
                        docker stop django-note-app || true
                        docker rm django-note-app || true
                        docker run -d -p 8000:8000 --name django-note-app ${env.dockerHubUser}/django-note-app:latest
                    EOF
                    """
                }
            }
        }
    }
}
