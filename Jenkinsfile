pipeline{
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t lanpir/nodeapp:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerhub')]){
                    sh "docker login -u lanpir -p ${dockerhub}"
                    sh "docker push lanpir/nodeapp:${DOCKER_TAG}"               
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['sshagent']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml root@192.168.120.230:/etc/kubernetes/yaml/nodeapp"
                    script{
                        try{
                            sh "ssh root@192.168.120.230 kubectl apply -f ."
                        }catch(error){
                            sh "ssh root@192.168.120.230 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}
