pipeline {
    agent { label 'ec2-agent' }

    environment {
        AWS_REGION   = 'us-west-2'
        ECR_REGISTRY = '744804011934.dkr.ecr.us-west-2.amazonaws.com'
        ECR_REPO     = 'infra-env-app'
        IMAGE_TAG    = "${BUILD_NUMBER}"
        CLUSTER_NAME = 'infra-env-cluster'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker build -t ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG} .
                    docker tag ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO}:latest
                """
            }
        }

        stage('Push To ECR') {
            steps {
                echo 'Pushing image to ECR...'
                sh """
                    docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ECR_REPO}:latest
                """
            }
        }

        stage('Deploy To EKS') {
            steps {
                echo 'Deploying to EKS...'
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                    kubectl set image deployment/app app=${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                    kubectl rollout status deployment/app
                """
            }
        }

    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
