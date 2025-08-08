pipeline {
  agent any

  environment {
    DOCKERHUB_CRED = 'dockerhub-creds'     // Jenkins credential ID for Docker Hub
    SSH_CRED = 'server-ssh-key'            // Jenkins credential ID for SSH
    IMAGE = 'vivekchindam21/sample'
    APP_NAME = 'account-service'
    APP_PORT = '8080'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Maven)') {
      steps {
        dir('account-service') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        dir('account-service') { // Go inside where Dockerfile exists
          sh "docker build -t ${IMAGE} ."
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CRED, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push ${IMAGE}
            docker logout
          '''
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        sshagent (credentials: [env.SSH_CRED]) {
          sh '''
            ssh -o StrictHostKeyChecking=no ec2-user@16.171.165.231'bash -s' <<'ENDSSH'
              set -e
              echo "Stopping existing container if present..."
              docker stop ${APP_NAME} || true
              docker rm ${APP_NAME} || true
              echo "Pulling image: ${IMAGE}"
              docker pull ${IMAGE}:latest
              echo "Starting container..."
              docker run -d --name ${APP_NAME} -p ${APP_PORT}:8080 --restart unless-stopped ${IMAGE}:latest
              echo "Deployment finished"
ENDSSH
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline completed successfully."
    }
    failure {
      echo "Pipeline failed. Check console output for details."
    }
  }
}
