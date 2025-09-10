pipeline {
  agent any
  environment {
    AWS_REGION = "${env.AWS_REGION ?: 'ap-south-1'}"
    ECR_URI = "${env.ECR_URI ?: '025066250063.dkr.ecr.ap-south-1.amazonaws.com/myproject'}"
    GITOPS_REPO = "https://github.com/your-org/gitops-repo.git"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Docker image') {
      steps {
        script {
          IMAGE_TAG = "build-${env.BUILD_NUMBER}"
          sh "docker build -t ${ECR_URI}:${IMAGE_TAG} ."
        }
      }
    }
    stage('Login to ECR') {
      steps {
        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}"
      }
    }
    stage('Push image to ECR') {
      steps {
        sh "docker push ${ECR_URI}:${IMAGE_TAG}"
      }
    }
    stage('Update gitops repo') {
      steps {
        withCredentials([string(credentialsId: 'GIT_TOKEN', variable: 'GH_TOKEN')]) {
          sh """
            rm -rf gitops || true
            git clone https://$GH_TOKEN@github.com/your-org/gitops-repo.git gitops
            cd gitops/k8s
            sed -i "s|IMAGE_PLACEHOLDER|${ECR_URI}:${IMAGE_TAG}|g" deployment.yaml
            git config user.email "jenkins@example.com"
            git config user.name "jenkins"
            git add deployment.yaml
            git commit -m "ci: update image to ${IMAGE_TAG}"
            git push origin main
          """
        }
      }
    }
  }
  post {
    success { echo "Pipeline success" }
    failure { echo "Pipeline failed" }
  }
}
