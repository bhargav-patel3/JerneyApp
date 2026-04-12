pipeline {
    agent any

    environment {
        IMAGE_NAME = "jerney-"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('git pull') {
            steps {
                git branch: 'main', url: 'https://github.com/bhargav-patel3/JerneyApp.git'
            }
        }

        stage('docker build') {
            steps {
                sh """
                docker build -t bunny9988/${IMAGE_NAME}frontend:${TAG} ./frontend
                docker build -t bunny9988/${IMAGE_NAME}backend:${TAG} ./backend
                """
            }
        }

        stage('docker login and push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    passwordVariable: 'DOCKER_PASS',
                    usernameVariable: 'DOCKER_USER'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push bunny9988/${IMAGE_NAME}frontend:${TAG}
                    docker push bunny9988/${IMAGE_NAME}backend:${TAG}
                    """
                }
            }
        }

        stage('Update k8s image name') {
            steps {
               sh """
                sed -i "s|jerney-backend:.*|jerney-backend:${BUILD_NUMBER}|g" kubernetes/backend-deployment.yml
                sed -i "s|jerney-frontend:.*|jerney-frontend:${BUILD_NUMBER}|g" kubernetes/frontend-deployment.yml
                """
                
                sh """
                cat kubernetes/backend-deployment.yml
                cat kubernetes/frontend-deployment.yml
                """
            }
        }

        stage('Deploy to EKS using k8s') {
            steps {
                sh """
                kubectl apply -f kubernetes/namespace.yml
                """
                sh """
                kubectl apply -f kubernetes/storageclass.yml
                kubectl apply -f kubernetes/pvc.yml
                kubectl apply -f kubernetes/secret.yml
                """
                sh """
                kubectl apply -f kubernetes/db-deployment.yml
                kubectl apply -f kubernetes/db-service.yml
                """

                sh """
                kubectl apply -f kubernetes/backend-deployment.yml
                kubectl apply -f kubernetes/frontend-deployment.yml
                """
                sh """
                kubectl apply -f kubernetes/backend-service.yml
                kubectl apply -f kubernetes/frontend-service.yml
                """

                sh """
                kubectl apply -f kubernetes/ingress.yml
                """
                sh """
                kubectl get all -n jerney-namespace
                """
            }
        }

        stage('verify Rollout') {
            steps {
                sh """
                kubectl rollout status deployment/jerney-backend -n jerney-namespace --timeout=120s
                kubectl rollout status deployment/jerney-frontend -n jerney-namespace --timeout=120s
                """
            }
        }
    }
}