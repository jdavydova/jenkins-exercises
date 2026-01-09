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

    Created [Dockerfile](Dockerfile)

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

    Test in browser.
    Open:

        http://localhost:3000


    
    
