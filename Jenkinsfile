pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-log')
        KUBE_TOKEN = credentials('kube-token-dev')
        KUBE_SERVER = 'https://$(minikube ip):8443'
        KUBE_NAMESPACE = 'dev'
        DOCKER_IMAGE = 'doaahemaid01/my-app:1.0'
        DEPLOYMENT_NAME = 'my-app'
    }
    stages {
        stage('Build Image') {
            steps {
                echo 'Building Docker image...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-log',
                                  usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        docker build -t \$DOCKER_IMAGE .
                        echo \$DOCKER_PASS | docker login --username \$DOCKER_USER --password-stdin
                        docker push \$DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to Dev Namespace') {
            steps {
                echo 'Deploying to Dev namespace...'
                sh """
                    kubectl config set-credentials jenkins --token=\$KUBE_TOKEN
                    kubectl config set-context jenkins-context --cluster=minikube --namespace=\$KUBE_NAMESPACE --user=jenkins
                    kubectl config use-context jenkins-context
                    if kubectl get deployment/\$DEPLOYMENT_NAME -n \$KUBE_NAMESPACE; then
                        kubectl set image deployment/\$DEPLOYMENT_NAME \$DEPLOYMENT_NAME=\$DOCKER_IMAGE -n \$KUBE_NAMESPACE
                    else
                        kubectl create deployment \$DEPLOYMENT_NAME --image=\$DOCKER_IMAGE -n \$KUBE_NAMESPACE
                    fi
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
