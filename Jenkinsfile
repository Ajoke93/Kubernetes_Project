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
        stage('Sending Dockerfile to Ansible Server over SSH') {
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "mkdir -p /home/ubuntu/Kubernetes_Project"
                            scp -r /home/jenkins/workspace/Kunernetes-Project/* ubuntu@172.31.2.68:/home/ubuntu/Kubernetes_Project
                        '''
                    }
                }
            }
        }
        stage('Building Docker image') {
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker image build -t $JOB_NAME:v1.$BUILD_ID ."
                        '''
                    }
                }
            }
        }
        stage('Building Docker tagging') {
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker tag $JOB_NAME:v1.$BUILD_ID ajoke93/$JOB_NAME:v1.$BUILD_ID"
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker tag $JOB_NAME:v1.$BUILD_ID ajoke93/$JOB_NAME:LATEST"
                        '''
                    }
                }
            }
        }
        stage('Push Images to Dockerhub') {
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        withCredentials([string(credentialsId: 'Dockerhub-Cred', variable: 'Dockerhub-Cred')]) {
                            sh '''
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "docker login -u ajoke93 -p $Dockerhub-Cred"
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "docker image push ajoke93/$JOB_NAME:v1.$BUILD_ID"
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "docker image push ajoke93/$JOB_NAME:LATEST"
                            '''
                        }
                    }
                }
            }
        }
    }
}
