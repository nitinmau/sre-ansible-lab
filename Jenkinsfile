pipeline {
    agent any
    environment {
        // Update these to match your actual AWX setup
        AWX_URL = "http://192.168.0.104" // Use your AWX IP
        JOB_ID  = "16"                  // Get this from the AWX Template URL
    }
    stages {
        stage('Checkout') {
            steps {
                // Use your actual GitHub repo URL
                git branch: 'main', url: 'https://github.com/nitinmau/ansible-jenkins-lab.git'
            }
        }

        stage('Run Jenkins Playbook') {
            steps {
                // The -e flag ensures Ansible uses Python3 on the target VM
                sh 'ansible-playbook install-jenkins.yml -i hosts -e "ansible_python_interpreter=/usr/bin/python3"'
            }
        }

        stage('Trigger AWX OTEL Job') {
            steps {
                withCredentials([string(credentialsId: 'awx-token', variable: 'AWX_TOKEN')]) {
                    sh """
                    curl -k -X POST \
                    -H "Authorization: Bearer $AWX_TOKEN" \
                    -H "Content-Type: application/json" \
                    ${AWX_URL}/api/v2/job_templates/${JOB_ID}/launch/
                    """
                }
            }
        }
    }
}
