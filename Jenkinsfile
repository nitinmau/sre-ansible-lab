pipeline {
    agent any
    
    environment {
        // Ensure this matches your VM IP
        AWX_URL = "http://192.168.0.104"
        // Ensure these IDs match your AWX Project and Template
        OTEL_JOB_ID = "9"
        AWX_PROJECT_ID = "7" 
    }

    stages {
        stage('Checkout') {
            steps {
                // FIXED: Pointing to the repo that contains all your playbooks
                git branch: 'main', url: 'https://github.com/nitinmau/sre-ansible-lab.git'
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
    
    post {
        always {
            echo "Pipeline execution finished."
        }
        success {
            echo "SRE Observability Stack Deployed Successfully!"
        }
        failure {
            echo "Pipeline failed. Check the console logs for details."
        }
    }
}
