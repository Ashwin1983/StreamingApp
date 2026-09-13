pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '087666071191'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Git Commit: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                      --region ${AWS_REGION} | \
                    docker login \
                      --username AWS \
                      --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Build Auth') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG} \
                      ./backend/authService
                '''
            }
        }

        stage('Build Streaming') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG} \
                      -f backend/streamingService/Dockerfile \
                      .
                '''
            }
        }

        stage('Build Admin') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG} \
                      -f backend/adminService/Dockerfile \
                      .
                '''
            }
        }

        stage('Build Chat') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG} \
                      -f backend/chatService/Dockerfile \
                      .
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    env.ALB_DNS = sh(
                        script: '''
                            kubectl get ingress streamingapp-ingress \
                              -n streamingapp \
                              -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                ''',
                returnStdout: true
            ).trim()

            echo "ALB DNS: ${env.ALB_DNS}"
                }

                sh '''
                    docker build \
                      --build-arg REACT_APP_AUTH_API_URL="http://${ALB_DNS}/api" \
                      --build-arg REACT_APP_STREAMING_API_URL="http://${ALB_DNS}/api" \
                      --build-arg REACT_APP_STREAMING_PUBLIC_URL="http://${ALB_DNS}" \
                      --build-arg REACT_APP_ADMIN_API_URL="http://${ALB_DNS}/api/admin" \
                      --build-arg REACT_APP_CHAT_API_URL="http://${ALB_DNS}/api/chat" \
                      --build-arg REACT_APP_CHAT_SOCKET_URL="http://${ALB_DNS}" \
                      -t ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG} \
                      ./frontend
                '''
           }
       }

        stage('Push All Images') {
            steps {
                sh '''
                    docker push ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Images') {
            steps {
                sh '''
                    echo "Verifying ECR images..."

                    aws ecr describe-images \
                      --repository-name streaming-auth \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}

                    aws ecr describe-images \
                      --repository-name streaming-stream \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}

                    aws ecr describe-images \
                      --repository-name streaming-admin \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}

                    aws ecr describe-images \
                      --repository-name streaming-chat \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}

                    aws ecr describe-images \
                      --repository-name streaming-frontend \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}
                '''
            }
        }
    }

    post {
        success {
            echo "All StreamingApp images pushed successfully with tag: ${IMAGE_TAG}"
        }

        failure {
            echo "StreamingApp CI pipeline failed. Check the console output."
        }

        always {
            sh 'docker logout ${ECR_REGISTRY} || true'
        }
    }
}
