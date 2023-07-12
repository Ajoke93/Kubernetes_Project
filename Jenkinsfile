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
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68'
                        sh 'scp /home/jenkins/workspace/Kunernetes-Project/* ubuntu@172.31.2.68:/home/ubuntu/Kubernetes_Project'
                    }
                }
            }
        }
    }
}
