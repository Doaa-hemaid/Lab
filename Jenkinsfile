pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-log')
        DOCKER_IMAGE_BASE = 'doaahemaid01/my-app'
        IMAGE_TAG = "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmmss')}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_BASE}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Image') {
            steps {
                echo 'Building Docker image...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-log',
                                  usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        docker build -t $DOCKER_IMAGE .
                        echo $DOCKER_PASS | docker login --username $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Clean Local Images') {
            steps {
                echo 'Cleaning up local Docker images...'
                sh """
                    docker rmi $DOCKER_IMAGE || true
                """
            }
        }

        stage('Deploy to Dev Namespace') {
            steps {
                echo 'Deploying to Dev namespace...'
                withCredentials([string(credentialsId: 'k8s-dev-token', variable: 'api_token')]) {
                    sh """
                        kubectl --token $api_token --server https://192.168.49.2:8443 \
                        --insecure-skip-tls-verify=true --validate=false apply -f myapp.yaml
                    """
                }
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
