pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'git@github.com:<username>/<repo>.git'
            }
        }

        stage('Run Jenkins Playbook') {
            steps {
                sh 'ansible-playbook install-jenkins.yml -i hosts'
            }
        }

        stage('Trigger AWX OTEL Job') {
            steps {
                withCredentials([string(credentialsId: 'awx-token', variable: 'AWX_TOKEN')]) {
                    sh '''
                    curl -X POST -H "Authorization: Bearer $AWX_TOKEN" \
                    https://<AWX_URL>/api/v2/job_templates/<JOB_ID>/launch/
                    '''
                }
            }
        }
    }
}       

