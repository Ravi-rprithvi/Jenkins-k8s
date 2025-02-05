pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'qa', 'prod'], description: 'Select the deployment environment')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip tests during build')
    }

    environment {
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'my-eks-cluster'
        ECR_REPO = '123456789012.dkr.ecr.us-east-1.amazonaws.com/my-node-app'
        IMAGE_TAG = "${params.DEPLOY_ENV}-${BUILD_NUMBER}"
        NAMESPACE = "${params.DEPLOY_ENV}"  // Each environment has its own namespace
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git 'https://github.com/example-org/nodejs-app.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def awsLogin = sh(script: "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO", returnStdout: true)
                    sh """
                        docker build -t $ECR_REPO:$IMAGE_TAG .
                        docker push $ECR_REPO:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER"

                    def kubeManifest = "k8s/deployment-${params.DEPLOY_ENV}.yaml"
                    sh """
                        kubectl apply -f ${kubeManifest} --namespace=${NAMESPACE}
                    """
                }
            }
        }

        stage('Update Ingress') {
            steps {
                script {
                    def ingressManifest = "k8s/ingress-${params.DEPLOY_ENV}.yaml"
                    sh """
                        kubectl apply -f ${ingressManifest} --namespace=${NAMESPACE}
                    """
                }
            }
        }
    }
}
