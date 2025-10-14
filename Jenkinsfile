pipeline {
    agent any
    
    environment {
        // Define environment variables
        APP_NAME = 'flask-docker-app'
        DOCKER_IMAGE = 'my-flask-app'
        TARGET_HOST = '13.232.242.245'  // Your target instance IP
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace and checkout code
                checkout scm
                sh 'git branch'
            }
        }
        
        stage('Code Quality Check') {
            steps {
                script {
                    // Simple code quality checks
                    sh '''
                        echo "Checking Python syntax..."
                        python -m py_compile app.py
                        echo "Checking YAML syntax..."
                        python -c "import yaml; yaml.safe_load(open('inventory.ini'))"
                    '''
                }
            }
        }
        
        stage('Test Ansible Syntax') {
            steps {
                script {
                    // Test Ansible playbook syntax
                    sh 'ansible-playbook --syntax-check deploy-docker-app.yml -i inventory.ini'
                }
            }
        }
        
        stage('Deploy to Target') {
            steps {
                script {
                    // Use SSH agent to deploy
                    sshagent(['target-server-ssh']) {
                        sh '''
                            # Deploy using Ansible
                            ansible-playbook -i inventory.ini deploy-docker-app.yml
                        '''
                    }
                }
            }
        }
        
        stage('Test Deployment') {
            steps {
                script {
                    // Test if application is running
                    sh '''
                        echo "Testing application endpoint..."
                        sleep 10  # Wait for app to start
                        curl -f http://${TARGET_HOST}:5000 || exit 1
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            // Notifications on success
            emailext (
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The pipeline ${env.BUILD_URL} completed successfully.",
                to: "leeshalulu01@gmail.com"
            )
        }
        failure {
            // Notifications on failure
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The pipeline ${env.BUILD_URL} failed. Please check the logs.",
                to: "leeshalulu01@gmail.com"
            )
        }
    }
}
