pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "nassimrafrafi/jenkins-docker-crud"
    }

    triggers {
        githubPush()   // Trigger uniquement sur push Webhook
    }

    stages {
        stage('Checkout Git') {
            steps {
                git branch: 'main', url: 'https://github.com/nassimrafrafi/devops.git'
            }
        }

        stage('Clean & Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Login Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-crud',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Clean images') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:latest || true"
            }
        }
    }

    post {
        success {
            echo "🔥 Pipeline Successful — Docker image pushed!"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
