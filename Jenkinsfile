@Library('jenkins-shared-library') _

pipeline {
  agent any
  stages {
    stage('Docker') {
      steps {
        script {
          dockerLogin(env.DOCKER_USER, env.DOCKER_PASS)
          buildImage("juliadavydova/my-app:${env.BUILD_NUMBER}")
          dockerPush("juliadavydova/my-app:${env.BUILD_NUMBER}")
        }
      }
    }
  }
}