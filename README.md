#### This project is for the DevOps bootcamp exercise for

#### "Build Automation with Jenkins"

##### Test
The project uses jest library for tests. (see "test" script in package.json)
There is 1 test (server.test.js) in the project that checks whether the main index.html file exists in the project. 
To run the nodejs test:

    npm run test

Make sure to download jest library before running test, otherwise jest command defined in package.json won't be found.

    npm install

In order to see failing test, remove index.html or rename it and run tests.

🔸 [EXERCISE 1: Dockerize your NodeJS App](Dockerfile)

    Configure your application to be built as a Docker image.

    Dockerize your NodeJS app

        git clone https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/jenkins-exercises.git
        cd jenkins-exercises 
        rm -rf .git
        git init
        git add . 
        git commit -m "first commit"
        git remote add origin git@github.com:jdavydova/jenkins-exercises.git
        git push -u origin main
        
   Create [Dockerfile](https://github.com/jdavydova/jenkins-exercises/blob/main/Dockerfile)

        FROM node:20-alpine

        # Create app directory
        RUN mkdir -p /usr/app
        COPY app/ /usr/app/

        WORKDIR /usr/app
        EXPOSE 3000

        RUN npm install
        CMD ["node", "server.js"]

    Run :

        docker build -t my-node-app .
        docker run -d -p 3000:3000 --name my-node-app my-node-app

    Test in browser](http://localhost:3000)
    Open:

        http://localhost:3000

🔸 [EXERCISE 2: Create a full pipeline for your NodeJS App]

You want the following steps to be included in your pipeline:

Increment version
The application's version and docker image version should be incremented.

TIP: Ensure to add —no-git-tag-version to the npm version minor command in your Jenkinsfile to avoid any commit errors

Run tests
You want to test the code, to be sure to deploy only working code. When tests fail, the pipeline should abort.

Build docker image with incremented version
Push to Docker repository
Commit to Git
The application version increment must be committed and pushed to a remote Git repository.

### Create Jenkins Credentials

Create usernamePassword credentials for docker registry called docker-credentials

Create usernamePassword credentials for git repository called github-credentials

### Create Jenkinsfile

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


    
## Notes:  \

✅ Option A (RECOMMENDED): Install Docker client via docker.io

This is the simplest and fastest way.

1️⃣ Create / edit Dockerfile

    FROM jenkins/jenkins:lts

    USER root
    RUN apt-get update && \
        apt-get install -y docker.io curl libatomic1 && \
        rm -rf /var/lib/apt/lists/*

    USER jenkins

2️⃣ Build the image

Run on the host, in the directory with the Dockerfile:

    docker build -t jenkins-docker .

3️⃣ Recreate Jenkins container (IMPORTANT)

You must remove the old container — otherwise changes do nothing.

    docker stop jenkins
    docker rm jenkins


Run Jenkins with Docker socket mounted:

    docker run -d --name jenkins \
      -p 8080:8080 -p 50000:50000 \
      -v jenkins_home:/var/jenkins_home \
      -v /var/run/docker.sock:/var/run/docker.sock \
      jenkins-docker

4️⃣ Verify (this MUST work)

    docker exec -it jenkins bash -lc "which docker && docker version"


Expected output:

    /usr/bin/docker

Docker client version info

<img width="1284" height="804" alt="Screenshot 2026-01-10 at 12 53 15 PM" src="https://github.com/user-attachments/assets/9fcf8456-2656-4a85-8632-bac541d51706" />

<img width="1460" height="843" alt="Screenshot 2026-01-10 at 1 04 54 PM" src="https://github.com/user-attachments/assets/48a0e793-94af-4c98-af65-64ea657802b5" />

<img width="952" height="568" alt="Screenshot 2026-01-10 at 1 03 56 PM" src="https://github.com/user-attachments/assets/6a582a25-1433-4b4a-bef6-6e097d249185" />



🔸 [EXERCISE 3: Manually deploy new Docker Image on server]

After the pipeline has run successfully, you:

Manually deploy the new docker image on the droplet server.


    ssh root@164.92.182.1

Add to Inbound Rules port 3000:

 <img width="858" height="240" alt="Screenshot 2026-01-12 at 10 32 28 AM" src="https://github.com/user-attachments/assets/187b00af-e0b0-4fc4-8457-48971126abef" />


To sign in with credentials on the command line, use 'docker login -u <username>'

    root@ubuntu-s-2vcpu-4gb-fra1-01:~# docker login -u juliadavydova

    docker run -d --name my-app \
      -p 3000:3000 \
      --restart unless-stopped \
      juliadavydova/my-app:1.1.0-8

    docker logs -f my-app

Open browser:

    http://164.92.182.1:3000/

<img width="873" height="653" alt="Screenshot 2026-01-12 at 10 33 58 AM" src="https://github.com/user-attachments/assets/d1d2a602-2c45-43ec-88a3-beab6e89cba0" />

🔸 [EXERCISE 4: Extract into Jenkins Shared Library]


A colleague from another project tells you that they are building a similar Jenkins pipeline and they could use some of your logic. So you suggest creating a Jenkins Shared Library to make your Jenkinsfile code reusable and shareable.

Therefore, you do the following:

Extract all logic into Jenkins-shared-library with parameters and reference it in Jenkinsfile.

Jenkinsfile:

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
    
            stage('Build and Push docker image') {
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


<img width="1009" height="585" alt="Screenshot 2026-01-13 at 10 56 51 AM" src="https://github.com/user-attachments/assets/a97c7c0a-41a8-4260-bdd0-4254072ed125" />






