pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Jenkins (Optional)') {
            steps {
                sh '''
                ansible-playbook -i hosts install-jenkins.yml
                '''
            }
        }

        stage('Install OTel') {
            steps {
                sh '''
                ansible-playbook -i hosts otel/otel-install.yml
                '''
            }
        }
    }
}


