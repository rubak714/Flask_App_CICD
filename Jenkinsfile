pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-cicd-app"
    CONTAINER_NAME = "flask_app"
  }

  stages {
    stage('Build Docker Image') {
      steps {
        sh "docker build -t $IMAGE_NAME ."
      }
    }

    stage('Stop Existing Container') {
      steps {
        sh "docker rm -f $CONTAINER_NAME || true"
      }
    }

    stage('Run Docker Container') {
      steps {
        sh "docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME"
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline completed!'
    }
    failure {
      echo '❌ Pipeline failed!'
    }
  }
}
