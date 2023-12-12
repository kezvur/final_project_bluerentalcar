pipeline {
    agent any
    environment {
        APP_NAME="car-rental"
        APP_STACK_NAME="brc-app-prod"
        APP_REPO_NAME="techpro-brc"
        AWS_REGION="us-east-1"
        AWS_ACCOUNT_ID=sh(script: 'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout: true).trim()
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    stages {
        stage('Prepare Tags for Docker Images') {
            steps {
                echo 'Preparing Tags for Docker Images'
                script {
                    env.IMAGE_TAG_FE = "${ECR_REGISTRY}/${APP_REPO_NAME}:frontend-prod-ver${BUILD_NUMBER}"
                    env.IMAGE_TAG_BE = "${ECR_REGISTRY}/${APP_REPO_NAME}:backend-prod-ver${BUILD_NUMBER}"
                    env.IMAGE_TAG_GR = "${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service-ver${BUILD_NUMBER}"
                    env.IMAGE_TAG_PR = "${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service-ver${BUILD_NUMBER}"
                }
            }
        }
        stage('Build App Docker Images') {
            steps {
                echo 'Building App Dev Images'
                sh """
                    docker build --force-rm -t "${IMAGE_TAG_FE}" "${WORKSPACE}/bluerentalcars-frontend"
                    docker build --force-rm -t "${IMAGE_TAG_BE}" "${WORKSPACE}/bluerentalcars-backend"
                    docker build --force-rm -t "${IMAGE_TAG_GR}" "${WORKSPACE}/grafana"
                    docker build --force-rm -t "${IMAGE_TAG_PR}" "${WORKSPACE}/prometheus"
                    docker image ls
                """
            }
        }
        stage('Create ECR Repository') {
            steps {
                echo "Creating ECR Repository ${APP_REPO_NAME}"
                sh """
                    aws ecr describe-repositories --repository-names "${APP_REPO_NAME}" --region ${AWS_REGION} || \
                    aws ecr create-repository --repository-name "${APP_REPO_NAME}" --region ${AWS_REGION}
                """
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker push "${IMAGE_TAG_FE}"
                    docker push "${IMAGE_TAG_BE}"
                    docker push "${IMAGE_TAG_GR}"
                    docker push "${IMAGE_TAG_PR}"
                """
            }
        }
        stage('Create K8s Namespace') {
            steps {
                echo 'Creating Kubernetes Namespace'
                sh "kubectl get ns brc || kubectl create ns brc"
            }
        }
        stage('Deploy App on EKS') {
            steps {
                echo 'Deploying App on K8s EKS Cluster'
                sh "envsubst < k8s/kustomization-template.yaml > k8s/kustomization.yaml"
                sh "kubectl apply -k k8s/"
            }
        }
        stage('Apply Ingress Controller Configuration') {
            steps {
                echo 'Applying Ingress Controller Configuration'
                sh "kubectl apply -f ${WORKSPACE}/ingress-controller.yaml"
            }
        }
        stage('Destroy the infrastructure') {
            steps {
                timeout(time: 5, unit: 'DAYS') {
                    input message: 'Approve terminate'
                }
                script {
                    // Delete application services and deployments in the 'brc' namespace
                    sh 'kubectl delete svc,deploy --all -n brc'
                    // Delete the Ingress controller's deployment
                    sh 'kubectl delete deployment ingress-nginx-controller -n ingress-nginx'
                    // Delete the entire 'ingress-nginx' namespace, which includes all related resources
                    sh 'kubectl delete namespace ingress-nginx'
                    
                    // Prune all local Docker images
                    sh 'docker image prune -af'
                    
                    // Delete the ECR repository (this will remove all images in the repository)
                    sh "aws ecr delete-repository --repository-name ${APP_REPO_NAME} --region ${AWS_REGION} --force"
                }
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
        failure {
            echo 'Delete the Image Repository on ECR due to the Failure'
            sh "aws ecr delete-repository --repository-name ${APP_REPO_NAME} --region ${AWS_REGION} --force"
        }
    }
}
