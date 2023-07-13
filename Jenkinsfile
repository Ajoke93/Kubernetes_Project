pipeline {
    agent {
        label 'Jenkins-slave1'
    }   
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git 'https://github.com/Ajoke93/Kubernetes_Project.git'
            }
        }
        stage('Moving Dockerfile to Jenkins') {
            sshagent(['ansible-cred']) {
                sh 'ssh -o StrictHostKeyChecking=no -l ubuntu@172.31.2.68'
                scp 'scp /home/jenkins/workspace/Kunernetes-Project/* ubuntu@172.31.2.68:/home/ubuntu'
            }
        }
    }
        stage('Buiding Docker file') {
            sshagent(['ansible-cred']) {
                sh 'ssh -o StrictHostKeyChecking=no -l ubuntu@172.31.2.68 cd /home/ubuntu/Kubernetes-Project'
                /* groovylint-disable-next-line LineLength */
                sh 'ssh -o StrictHostKeyChecking=no -l ubuntu@172.31.2.68 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
            }
}
