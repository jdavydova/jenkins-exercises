@Library('jenkins-shared-library') _

pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "juliadavydova/my-app"
    }
    tools {
        nodejs "my-nodejs"
    }
    stages {
        stage('increment version') {
            agent {
                docker {
                    image 'node:20-bullseye'
                    // optional: avoids permission issues on workspace files
                    args '-u root:root'
                }
            }
            steps {
                dir("app") {
                    sh "npm ci"
                    sh "npm version minor --no-git-tag-version"

                    script {
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version
                        env.IMAGE_NAME = "${version}-${env.BUILD_NUMBER}"
                        echo "IMAGE_NAME set to: ${env.IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Docker') {
            steps {
                script {
                    def image = "${DOCKER_IMAGE}:${env.IMAGE_NAME}"
                    dockerLogin('docker-credentials')
                    buildImage(image)
                    dockerPush(image)
                }
            }
        }
    }
}
