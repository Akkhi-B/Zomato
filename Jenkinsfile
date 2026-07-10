pipeline {
    agent any

    environment {
        AWS_REGION            = 'ap-south-1'
        ECR_REPO              = '700918784883.dkr.ecr.ap-south-1.amazonaws.com/zomato'
        IMAGE_TAG             = "v${BUILD_NUMBER}"
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Akkhi-B/Zomato.git'
            }
        }

        stage('npm Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t zomato:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_REPO}
                docker tag zomato:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                aws eks update-kubeconfig --name zomato-cluster-2 --region ${AWS_REGION}
                kubectl apply -f k8s-deployment.yml
                kubectl set image deployment/zomato-deployment zomato=${ECR_REPO}:${IMAGE_TAG}
                kubectl apply -f k8s-service.yml
                kubectl rollout status deployment/zomato-deployment
                """
            }
        }
    }

    post {
        success { echo 'Deployment successful' }
        failure { echo 'Pipeline failed' }
    }
}
