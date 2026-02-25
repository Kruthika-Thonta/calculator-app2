pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kruthi2008/my-k8s-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kruthika-Thonta/calculator-app2.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                npm install
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                '''
            }
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push $DOCKER_IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('Start Minikube (if not running)') {
            steps {
                sh '''
                minikube start --driver=docker
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Replace image tag inside deployment.yaml
                kubectl set image deployment/my-k8s-app-deployment \
                my-k8s-app=$DOCKER_IMAGE:$IMAGE_TAG

                # Apply Kubernetes files
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl rollout status deployment/my-k8s-app-deployment
                '''
            }
        }
    }
}