pipeline {
    agent {
        label 'Jenkins-slave1'
    }
   
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git 'https://github.com/Ajoke93/Kubernetes_Project.git'
            }
        }
    }
}
