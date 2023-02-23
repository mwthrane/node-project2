pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        
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
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'hub.docker', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t mwthrane/demo-app:NODEJS-2.0 ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push mwthrane/demo-app:NODEJS-2.0"
                }
            }
        }

    }
}

