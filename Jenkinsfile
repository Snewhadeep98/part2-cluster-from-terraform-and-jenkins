pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "eu-west-3"
        registry = "871154751090.dkr.ecr.eu-west-2.amazonaws.com/aggregator-service"
    }
    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'release/preprod', credentialsId: '06ca9a02-ed80-432e-8918-9fb25359c39d', url: 'git@bitbucket.org:ncuk/ncuk_aggregator_service.git'
            }
        }
        // Building Docker images
        stage('Building image') {
            steps {
                script {
                    // Use docker buildx to build the image with the appropriate tag
                      sh 'docker build -t 871154751090.dkr.ecr.eu-west-2.amazonaws.com/aggregator-service:latest .'
                }
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 871154751090.dkr.ecr.eu-west-2.amazonaws.com'
                    sh 'docker push 871154751090.dkr.ecr.eu-west-2.amazonaws.com/aggregator-service:latest'
                }
            }
        }
    // Added a closing brace for stages block
 // Added a closing brace for pipeline block

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
                        sh "kubectl apply -f aggregator-deployment.yaml"
                        sh "kubectl apply -f aggregator-service.yaml"
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
                        sh "kubectl apply -f 03_HPA.yml"

                    }
                }
            }
        }
    }
}