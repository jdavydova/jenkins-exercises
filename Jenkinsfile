pipeline {
    agent any

    tools {
        nodejs "my-nodejs"
    }

    environment {
        DOCKER_IMAGE = "juliadavydova/my-app"
    }

    stages {

        stage('increment version') {
            steps {
                dir("app") {
                    // install deps so npm can update package-lock.json correctly (and scripts can run)
                    sh "npm install"

                    // IMPORTANT: use two normal hyphens, not the long dash character
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

        stage('Run tests') {
            steps {
                dir("app") {
                    sh "npm test"
                }
            }
        }

        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        set -e
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE}:${IMAGE_NAME} .
                        docker push ${DOCKER_IMAGE}:${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('commit version update') {
             steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        set -e

                        git config user.email "jenkins@example.com"
                        git config user.name "jenkins"

                         git remote set-url origin https://$USER:$PASS@github.com/jdavydova/jenkins-exercises.git

                         git add app/package.json app/package-lock.json || true
                         git diff --cached --quiet || git commit -m "ci: version bump to ${IMAGE_NAME}"

                         git push origin HEAD:jenkins-jobs
                    '''
                }
             }
        }
    }
}
