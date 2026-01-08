# terraform-jenkins

pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {

        stage('Checkout Terraform Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/1808-me/terraform-jenkins.git'
            }
        }

        stage('Terraform Init') {
            steps {
                sh '''
                  terraform init
                '''
            }
        }

        stage('Terraform Plan') {
            steps {
                sh '''
                  terraform plan -no-color | tee tfplan.txt
                '''
            }
        }

        stage('Approve Infrastructure Deployment') {
            steps {
                input message: 'Do you want to APPLY this Terraform plan to AWS?',
                      ok: 'Yes, deploy infrastructure'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh '''
                  terraform apply -auto-approve
                '''
            }
        }
    }

    post {
        success {
            echo 'Terraform pipeline completed successfully'
        }
        failure {
            echo 'Terraform pipeline failed'
        }
    }
}
