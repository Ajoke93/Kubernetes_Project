pipeline {
    agent {
        label 'agent-2'
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
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "mkdir -p /home/ubuntu/kubernetes_project"
                            scp -r /home/ubuntu/workspace/kubernetes-project/* ubuntu@172.31.2.68:/home/ubuntu/kubernetes_project
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
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/kubernetes_project && docker image build -t ${JOB_NAME}:v1.${BUILD_ID} ."
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
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/kubernetes_project && docker tag ${JOB_NAME}:v1.${BUILD_ID} ajoke93/${JOB_NAME}:v1.${BUILD_ID}"
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 "cd /home/ubuntu/kubernetes_project && docker tag ${JOB_NAME}:v1.${BUILD_ID} ajoke93/${JOB_NAME}:LATEST"
                        '''
                    }
                }
            }
        }
        stage('Push Images to Dockerhub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sshagent(['ansible-cred']) {
                            sh """
                                echo \$DOCKERHUB_PASSWORD | docker login -u \$DOCKERHUB_USERNAME --password-stdin
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 'docker image push ajoke93/${JOB_NAME}:v1.${BUILD_ID}'
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.2.68 'docker image push ajoke93/${JOB_NAME}:LATEST'
                            """
                        }
                    }
                }
            }
        }
        stage('Copy file from Ansible to Kubernetes cluster') {
            steps {
                script {
                    sshagent(['kubernetes_server']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.1.140 'mkdir -p /home/ubuntu/kubernetes_project'
                            scp -r /home/ubuntu/workspace/kubernetes-project/* ubuntu@172.31.1.140:/home/ubuntu/kubernetes_project
                        """
                    }
                }
            }
        }
        stage('Kubernetes Deployment using ansible') {
            steps {
                script {
                    sshagent(['ansible-cred']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@172.31.1.140 "ansible-playbook -i inventory.yml /home/ubuntu/kubernetes_project/ansible.yml"
                        """
                    }
                }
            }
        }
    }
}
