pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "eu-west-3"
    }
    stages {
        stage("Create an EKS Cluster") {
            steps {
                script {
                    dir('terraform-for-cluster') {
                        sh "terraform init"
                        sh "terraform validate"
                        sh "terraform plan"
                        sh "terraform apply -auto-approve"
                    }
                }
            }
        }
        stage("Deploy to EKS") {
            steps {
                script {
                    dir('kubernetes') {
                        sh "aws eks update-kubeconfig --name myjenkins-server-eks-cluster --region eu-west-3"
                        sh "kubectl apply -f deployment.yaml"
                        sh "kubectl apply -f service.yaml"
                    }
                }
            }
        }
         stage("Metrics Server Installation") {
            steps {
                script {
                    dir('kubernetes') {
                        sh "kubectl apply -f metrics-apiservice.yaml"
                        sh "kubectl apply -f metrics-rbac.yaml"
                        sh "kubectl apply -f metrics-server-deployment.yaml"
                        sh "kubectl apply -f metrics-server-service.yaml"
                        sh "kubectl apply -f metrics-serviceaccount.yaml"
                    }
                }
            }
        }
        stage("Configure Auto-Scaling") {
            steps {
                script {
                    dir('kubernetes') {
                        sh "kubectl apply -f HPA.yml"

                    }
                }
            }
        }
    }
}