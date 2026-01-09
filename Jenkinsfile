pipeline {
  agent any

  environment {
    AWS_REGION = "eu-west-2"
    AWS_ACCOUNT_ID = "968726368728"
    ECR_REPO = "hello-eks"
    EKS_CLUSTER = "real-eks-demo"
    IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build') {
      steps {
        sh '''
          set -e
          TAG=$(git rev-parse --short HEAD)
          echo $TAG > .tag
          docker build -t ${ECR_REPO}:$TAG .
          docker tag ${ECR_REPO}:$TAG ${IMAGE_URI}:$TAG
        '''
      }
    }

    stage('ECR Login') {
      steps {
        sh '''
          set -e
          aws ecr get-login-password --region ${AWS_REGION} | \
          docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        '''
      }
    }

    stage('Push') {
      steps {
        sh '''
          set -e
          TAG=$(cat .tag)
          docker push ${IMAGE_URI}:$TAG
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -e
          TAG=$(cat .tag)
          aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
          kubectl set image deployment/hello-eks hello-eks=${IMAGE_URI}:$TAG
          kubectl rollout status deployment/hello-eks
        '''
      }
    }
  }
}
