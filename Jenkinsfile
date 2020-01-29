pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t dmedema/node-app:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                    sh "docker login -u dmedema -p ${dockerHubPwd}"
                    sh "docker push dmedema/node-app:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to dev-server'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['aws-ec2-user']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@54.219.162.180:/home/ec2-user/"
                    script{
                        try{
                            sh "ssh ec2-user@54.219.162.180 kubectl apply -f ."
                        }catch(error){
                            sh "ssh ec2-user@54.219.162.180 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
