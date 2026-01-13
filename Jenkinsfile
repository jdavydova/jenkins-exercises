@Library('jenkins-shared-library') _

pipeline {
  agent any
  stages {
    stage('Docker') {
      steps {
        script {
          def image = "juliadavydova/my-app:${env.BUILD_NUMBER}"
          dockerLogin('docker-credentials')
          buildImage(image)
          dockerPush(image)
        }
      }
    }
  }
}