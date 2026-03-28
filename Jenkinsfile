pipeline {
    agent any
    environment {
        AWX_URL = "http://192.168.0.104"
        OTEL_JOB_ID = "9" // Ensure this is the ID of your OTEL Template in AWX
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nitinmau/ansible-jenkins-lab.git'
            }
        }
        stage('Self-Repair & Deploy Jenkins') {
            steps {
                // Runs the RAW playbook you just showed me to fix the Python/Jenkins environment
                sh 'ansible-playbook install-jenkins.yml -i hosts'
            }
        }
        stage('Trigger AWX OTel') {
            steps {
                // Uses the 'awx-token' you already configured in Jenkins
                withCredentials([string(credentialsId: 'awx-token', variable: 'AWX_TOKEN')]) {
                    sh """
                    curl -k -X POST \
                    -H "Authorization: Bearer ${AWX_TOKEN}" \
                    -H "Content-Type: application/json" \
                    ${AWX_URL}/api/v2/job_templates/${OTEL_JOB_ID}/launch/
                    """
                }
            }
        }
    }
}
