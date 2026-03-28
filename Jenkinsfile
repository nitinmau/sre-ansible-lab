pipeline {
    agent any

    environment {
        AWX_URL = "http://192.168.0.104"
        OTEL_JOB_ID = "9"
        AWX_PROJECT_ID = "7"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nitinmau/sre-ansible-lab.git'
            }
        }

        // --- SKIPPED JENKINS INSTALL TO AVOID REDUNDANCY ---

        stage('Deploy Grafana') {
            steps {
                // ADDED: export ANSIBLE_HOST_KEY_CHECKING for every stage to be safe
                sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook install-grafana.yml -i hosts'
            }
        }

        stage('Deploy Prometheus') {
            steps {
                sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook install-prometheus.yml -i hosts'
            }
        }

        stage('Sync AWX Project') {
            steps {
                // SRE Tip: Using credentialsId ensures your token isn't leaked in logs
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
