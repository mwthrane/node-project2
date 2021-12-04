</details>

******

<details>
<summary>Exercise 1: Dockerize your NodeJS App </summary>
 <br />

**Dockerfile**
```sh
FROM node:13-alpine

RUN mkdir -p /usr/app
COPY app/* /usr/app/

WORKDIR /usr/app
EXPOSE 3000

RUN npm install
CMD ["node", "server.js"]

```

</details>

******

<details>
<summary>Exercise 2: Create a full pipeline for your NodeJS App </summary>
 <br />

**Create Jenkins Credentials**
- Create 

**Configure Node Tool in Jenkins Configuration**
- Name should be `node`, because that's how it's referenced in the below Jenkinsfile in `tools` block

**Install plugin**
- Install `Pipeline Utility Steps` plugin. This contains readJSON function, that we will use to read the version from package.json 

**Jenkinsfile**

```sh
pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    def packageJson = readJSON file: 'app/package.json'
                    env.IMAGE_NAME = "$packageJson.version-$BUILD_NUMBER"
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    dir("app") {
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Push docker image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t docker-hub-id/myapp:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push docker-hub-id/myapp:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        # git config here for the first time run
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/devops-bootcamp3/node-project.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}


```



</details>

******

<details>
<summary>Exercise 2, 3: Create and prepare server on Digital Ocean </summary>
 <br />

**steps:**
```sh
# ssh into your newly created server
ssh -i ~/id_rsa root@{server-ip-address}

# install nodejs and npm
apt install -y nodejs npm

```

</details>

******

<details>
<summary>Exercise 4: Copy App and package.json </summary>
 <br />

**steps:**
```sh
# secure copy files from local machine to server. Execute from project's root folder.
scp -i ~/id_rsa app/bootcamp-node-project-1.0.0.tgz root@{server-ip-address}:/root
scp -i ~/id_rsa app/package.json root@{server-ip-address}:/root

```

</details>

******

<details>
<summary>Exercise 5: Run Node App </summary>
 <br />

**steps:**
```sh
# ssh into droplet
ssh -i ~/id_rsa root@{server-ip-address}

# unpack the node-project file
tar zxvf bootcamp-node-project-1.0.0.tgz

# change into unpacked directory called "package"
cd package

# install dependencies
npm install

# run the application
node server.js

```

</details>

******
