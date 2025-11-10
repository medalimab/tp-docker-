pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    skipDefaultCheckout(true)   // évite le stage auto "Declarative: Checkout SCM"
  }

  environment {
    // ==== À adapter si besoin ====
    REPO_URL  = 'https://github.com/medalimab/tp-docker-.git'
    BRANCH    = 'main'

    DOCKERHUB_CREDENTIALS = 'dockerhub'    // ID des creds Docker Hub dans Jenkins
    IMAGE_SERVER = 'daliii/mern-server'
    IMAGE_CLIENT = 'daliii/mern-client'

    // optionnel : active BuildKit (build plus rapide)
    DOCKER_BUILDKIT = '1'
  }

  stages {

    stage('Checkout Code') {
      steps {
        deleteDir() // workspace propre
        git branch: env.BRANCH, url: env.REPO_URL
      }
    }

    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: env.DOCKERHUB_CREDENTIALS,
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          '''
        }
      }
    }

    stage('Build & Push Server Image') {
      when { expression { return fileExists('server/Dockerfile') } }
      steps {
        sh '''
          docker build -t ${IMAGE_SERVER}:${BUILD_NUMBER} server
          docker push ${IMAGE_SERVER}:${BUILD_NUMBER}
          docker tag  ${IMAGE_SERVER}:${BUILD_NUMBER} ${IMAGE_SERVER}:latest
          docker push ${IMAGE_SERVER}:latest
        '''
      }
    }

    stage('Build & Push Client Image') {
      when { expression { return fileExists('client/Dockerfile') } }
      steps {
        sh '''
          docker build -t ${IMAGE_CLIENT}:${BUILD_NUMBER} client
          docker push ${IMAGE_CLIENT}:${BUILD_NUMBER}
          docker tag  ${IMAGE_CLIENT}:${BUILD_NUMBER} ${IMAGE_CLIENT}:latest
          docker push ${IMAGE_CLIENT}:latest
        '''
      }
    }
  }

  post {
    always {
      sh 'docker system prune -af || true'
    }
    success {
      echo "✅ Build terminé et images poussées sur Docker Hub avec succès !"
    }
    failure {
      echo "❌ Build échoué — vérifie la console Jenkins."
    }
  }
}
