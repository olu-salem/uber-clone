pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = credentials('account_id')
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }
    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/olu-salem/uber-clone.git'
            }
        }
        stage('Terraform version') {
            steps {
                sh 'terraform --version'
            }
        }
        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                    # Install Python virtual environment if not already installed
                    python3 -m venv venv
                    . venv/bin/activate
                    # Upgrade pip
                    pip install --upgrade pip
                '''
            }
        }
        stage('Install Checkov') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install checkov
                '''
            }
        }
        stage('Checkov scan') {
            steps {
                dir('EKS_Terraform') {
                    sh '''
                        . ../venv/bin/activate
                        checkov -d . --output-file-path checkov_report.json --quiet
                    '''
                }
            }
        }
        stage('Publish Checkov Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'EKS_Terraform',
                    reportFiles: 'checkov_report.json',
                    reportName: 'Checkov Report',
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }
        stage('Terraform init') {
            steps {
                dir('EKS_Terraform') {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform validate') {
            steps {
                dir('EKS_Terraform') {
                    sh 'terraform validate'
                }
            }
        }
        stage('Terraform plan') {
            steps {
                dir('EKS_Terraform') {
                    sh 'terraform plan'
                }
            }
        }
        stage('Terraform apply/destroy') {
            steps {
                dir('EKS_Terraform') {
                    sh 'terraform ${action} --auto-approve'
                }
            }
        }
    }
    post {
        always {
            sh 'rm -rf venv'
        }
    }
}
