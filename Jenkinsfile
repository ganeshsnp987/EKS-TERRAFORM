pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Terraform Action')
    }

    environment {
        AWS_CREDENTIALS = credentials('aws_cred')  // Injects AWS_CREDENTIALS_ACCESS_KEY_ID
                                                   // and AWS_CREDENTIALS_SECRET_ACCESS_KEY
    }

    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ganeshsnp987/EKS-TERRAFORM.git'
            }
        }

        stage('Terraform Version') {
            steps {
                sh 'terraform --version'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan'
            }
        }

        stage('Terraform Apply / Destroy') {
            steps {
                sh "terraform ${params.action} --auto-approve"
            }
        }
    }

    post {
        success {
            echo "✅ Terraform ${params.action} completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above."
        }
        always {
            cleanWs()  // Clean workspace after every run
        }
    }
}
