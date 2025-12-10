pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "nassimrafrafi/jenkins-docker-crud"
    DOCKER_CRED  = "docker-crud"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/nassimrafrafi/devops.git'
        script { echo "Branch: ${env.BRANCH_NAME ?: 'main'}" }
      }
    }

    stage('Clean + Build Maven') {
      steps {
        sh 'mvn -B -DskipTests=false clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Unit tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Docker build') {
      steps {
        script {
          def tag = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

          sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
          sh "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
        }
      }
    }

    stage('Docker push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            TAG=$(git rev-parse --short HEAD)

            docker push ${DOCKER_IMAGE}:$TAG
            docker push ${DOCKER_IMAGE}:latest

            docker logout
          '''
        }
      }
    }
  }

  post {
    success { echo "🔥 Pipeline succeeded and Docker image pushed to: ${DOCKER_IMAGE}" }
    failure { echo "❌ Pipeline failed" }
  }
}
