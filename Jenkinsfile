pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kruthi2008/my-k8s-app2"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
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

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # First apply deployment and service
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml

                # Then update image
                kubectl set image deployment/my-k8s-app-deployment \
                my-k8s-app=$DOCKER_IMAGE:$IMAGE_TAG

                
                # Wait for rollout
                kubectl rollout status deployment/my-k8s-app-deployment
                '''
            }
        }

        
    }
}