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
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker image build -t ${JOB_NAME}:v1.${BUILD_ID} ."
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
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker tag ${JOB_NAME}:v1.${BUILD_ID} ajoke93/${JOB_NAME}:v1.${BUILD_ID}"
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/Kubernetes_Project && docker tag ${JOB_NAME}:v1.${BUILD_ID} ajoke93/${JOB_NAME}:LATEST"
                        '''
                    }
                }
            }
        }
        stage('Push Images to Dockerhub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'ae64a27f-523c-4140-bd88-614d48e714d6', passwordVariable: 'docker_passwd', usernameVariable: 'docker_username')]) {
                        sshagent(['ansible-cred']) {
                            sh """
                                "docker login -u ${docker_username} -p ${docker_passwd}"
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 'docker image push ${docker_username}/${JOB_NAME}:v1.${BUILD_ID}'
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 'docker image push ${docker_username}/${JOB_NAME}:LATEST'
                            """
                        }
                    }
                }
            }
        }
    }
}
