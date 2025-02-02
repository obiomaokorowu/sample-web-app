pipeline {
    agent any

    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="111393898725"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="october-class"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"


    }
    stages {
          stage('Building docker image') {
            
            steps{
              script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
               }
            }
          }
        
          stage('Pushing to AWS ECR') {
             environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                         
                   }
              steps{  
                script {
                    sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                    sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                    sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
                        }
                    }
            }
          stage('pull image & Deploying application on k8s cluster') {
                    environment {
                       AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                       AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                 }
                    steps {
                      script{
                          sh 'aws eks update-kubeconfig --name app-eks-cluster --region us-east-2'
                          sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                          sh 'helm upgrade --install --namespace config --set image.repository="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}" --set image.tag="${VERSION}" october-app ./web-app -f ./web-app/values.yaml'



 
                        }
                    }
            }
    }
}
