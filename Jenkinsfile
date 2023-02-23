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
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {

                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh "git remote set-url origin https://${USER}:${PWD}@hgithub.com/mwthrane/node-project2.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}

