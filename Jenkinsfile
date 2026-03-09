pipeline {
    agent any

    environment {
        DOCKERHUB_REPO     = "deepakc742004/trend-app"
        IMAGE_TAG          = "${BUILD_NUMBER}"
        AWS_DEFAULT_REGION = "us-east-1"
        EKS_CLUSTER_NAME   = "trend-eks-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/deepak-c2004/trend-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKERHUB_REPO:$IMAGE_TAG -t $DOCKERHUB_REPO:latest .
                '''
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $DOCKERHUB_REPO:$IMAGE_TAG
                    docker push $DOCKERHUB_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

                        aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $EKS_CLUSTER_NAME

                        git checkout -- deployment.yaml
                        sed -i "s|IMAGE_PLACEHOLDER|$DOCKERHUB_REPO:$IMAGE_TAG|g" deployment.yaml

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl rollout status deployment/trend-app
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
