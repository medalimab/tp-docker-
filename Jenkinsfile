pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    skipDefaultCheckout(true)   // évite le stage auto "Declarative: Checkout SCM"
  }

  environment {
    REPO_URL  = 'https://github.com/medalimab/tp-docker-.git'
    BRANCH    = 'main'

    DOCKERHUB_CREDENTIALS = 'dockerhub'
    IMAGE_SERVER = 'daliii/mern-server'
    IMAGE_CLIENT = 'daliii/mern-client'

    DOCKER_BUILDKIT = '1'
  }

  stages {

    stage('Checkout Code') {
      steps {
        deleteDir()
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

    // ---------- TRIVY SCANS ----------
    stage('Trivy Scan (server)') {
      when { expression { return fileExists('server/Dockerfile') } }
      steps {
        sh '''
          # Scan CRITICAL,HIGH et sauvegarde du rapport
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image --no-progress \
            --severity CRITICAL,HIGH \
            ${IMAGE_SERVER}:${BUILD_NUMBER} | tee trivy_report_server.txt

          # ne pas faire échouer le job sur findings
          true
        '''
        archiveArtifacts artifacts: 'trivy_report_server.txt', onlyIfSuccessful: false
      }
    }

    stage('Trivy Scan (client)') {
      when { expression { return fileExists('client/Dockerfile') } }
      steps {
        sh '''
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image --no-progress \
            --severity CRITICAL,HIGH \
            ${IMAGE_CLIENT}:${BUILD_NUMBER} | tee trivy_report_client.txt
          true
        '''
        archiveArtifacts artifacts: 'trivy_report_client.txt', onlyIfSuccessful: false
      }
    }
  }

  post {
    always {
      sh 'docker system prune -af || true'
    }
    success {
      echo "✅ Build terminé, images poussées, et rapports Trivy archivés."
    }
    failure {
      echo "❌ Build échoué — vérifie la console Jenkins."
    }
  }
}
