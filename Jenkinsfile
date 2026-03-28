pipeline {
    agent any
    environment {
        AWX_URL = "http://192.168.0.104"
        OTEL_JOB_ID = "9"
        AWX_PROJECT_ID = "7" // Project ID for 'Ansible Jenkins Lab'
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
        stage('Deploy Prometheus') {
            steps {
                sh 'ansible-playbook install-prometheus.yml'
            }
        }
        // --- NEW SYNC STAGE ---
        stage('Sync AWX Project') {
            steps {
                withCredentials([string(credentialsId: 'awx-token', variable: 'AWX_TOKEN')]) {
                    sh """
                    curl -k -X POST \
                    -H "Authorization: Bearer ${AWX_TOKEN}" \
                    -H "Content-Type: application/json" \
                    ${AWX_URL}/api/v2/projects/${AWX_PROJECT_ID}/update/
                    """
                }
                echo "Waiting 15 seconds for AWX Sync to complete..."
                sleep 15
            }
        }
        // -----------------------
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
