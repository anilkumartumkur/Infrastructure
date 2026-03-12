pipeline {

    parameters {
        choice(
            name: 'terraformAction',
            choices: ['apply', 'destroy'],
            description: 'Choose your terraform action'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION    = 'us-west-2'
    }

    agent any

    stages {

        stage('Checkout') {
            steps {
                script {
                    dir('terraform') {
                        git url: 'https://github.com/ITkannadigaru/Infrastructure.git', branch: 'main'
                    }
                }
            }
        }

        stage('Plan: All Layers') {
            steps {
                sh 'cd terraform/0-bootstrap && terraform init -input=false && terraform plan -out tfplan && terraform show -no-color tfplan > tfplan.txt'
                sh 'cd terraform/1-network && terraform init -input=false && terraform plan -out tfplan && terraform show -no-color tfplan > tfplan.txt'
                sh 'cd terraform/2-eks && terraform init -input=false && terraform plan -out tfplan && terraform show -no-color tfplan > tfplan.txt'
            }
        }

        stage('Approval') {
            steps {
                script {
                    def bootstrap = readFile 'terraform/0-bootstrap/tfplan.txt'
                    def network   = readFile 'terraform/1-network/tfplan.txt'
                    def eks       = readFile 'terraform/2-eks/tfplan.txt'
                    def allPlans  = "=== 0-bootstrap ===\n${bootstrap}\n=== 1-network ===\n${network}\n=== 2-eks ===\n${eks}"
                    input message: 'Review all layer plans and approve to proceed',
                          parameters: [text(name: 'Plan', description: 'Terraform Plan Output', defaultValue: allPlans)]
                }
            }
        }

        stage('Execute: 0-bootstrap') {
            steps {
                script {
                    if (params.terraformAction == 'apply') {
                        sh 'cd terraform/0-bootstrap && terraform apply -input=false tfplan'
                    } else {
                        sh 'cd terraform/0-bootstrap && terraform destroy -auto-approve'
                    }
                }
            }
        }

        stage('Execute: 1-network') {
            steps {
                script {
                    if (params.terraformAction == 'apply') {
                        sh 'cd terraform/1-network && terraform apply -input=false tfplan'
                    } else {
                        sh 'cd terraform/1-network && terraform destroy -auto-approve'
                    }
                }
            }
        }

        stage('Execute: 2-eks') {
            steps {
                script {
                    if (params.terraformAction == 'apply') {
                        sh 'cd terraform/2-eks && terraform apply -input=false tfplan'
                    } else {
                        sh 'cd terraform/2-eks && terraform destroy -auto-approve'
                    }
                }
            }
        }

    }

    post {
        success {
            echo "All layers completed: terraform ${params.terraformAction} succeeded."
        }
        failure {
            echo "Pipeline failed. Check the stage logs above."
        }
        always {
            cleanWs()
        }
    }
}
