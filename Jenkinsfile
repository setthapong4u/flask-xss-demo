pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'setthapong/xs-app-demo'
        IMAGE_TAG    = 'latest'   
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                sh '''
                  echo "Building Docker image..."
                  docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    sh '''
                      echo "Logging in to Docker Hub..."
                      echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin

                      echo "Pushing image to Docker Hub..."
                      docker push ${DOCKER_IMAGE}:${IMAGE_TAG}

                      docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  echo "Applying Kubernetes manifests..."
                  kubectl apply -f k8s/deployment.yaml
                  kubectl apply -f k8s/service.yaml

                  echo "Current pods:"
                  kubectl get pods -o wide

                  echo "Current services:"
                  kubectl get svc -o wide
                '''
            }
        }
    }
}
