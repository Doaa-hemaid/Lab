pipeline {
    agent any

    stages {
        stage('Build Image') {
            steps {
                echo 'Building...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-log',
                                  usernameVariable: 'DEPLOY_USER', passwordVariable: 'DEPLOY_PASSWORD')]) 
                {
                    sh """
                        docker build -t doaahemaid01/my-app:1.0 .
                        echo $passwd | docker login --username $username --password-stdin
                        docker push doaahemaid01/my-app:1.0 
                    """
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add your deployment steps here
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
