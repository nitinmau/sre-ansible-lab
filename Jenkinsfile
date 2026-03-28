pipeline {
    agent any
    environment {
        AWX_URL = "http://192.168.0.104"
        OTEL_JOB_ID = "9"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nitinmau/ansible-jenkins-lab.git'
            }
        }
        stage('Self-Repair & Deploy Jenkins') {
            steps {
                sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook install-jenkins.yml -i hosts'
            }
        }
        stage('Deploy Grafana') {
            steps {
                sh 'ansible-playbook install-grafana.yml'
            }
        }
        // --- ADDED PROMETHEUS STAGE ---
        stage('Deploy Prometheus') {
            steps {
                sh 'ansible-playbook install-prometheus.yml'
            }
        }
        // ------------------------------
        stage('Trigger AWX OTel') {
            steps {
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
