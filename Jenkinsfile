pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKERHUB_REPO_DEV = 'ramprasad1090/react-app-dev'
        DOCKERHUB_REPO_PROD = 'ramprasad1090/react-app-prod'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "whoami"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub') {
                        if (env.BRANCH_NAME == 'dev') {
                            sh "docker tag ${DOCKERHUB_REPO_DEV}:${env.BRANCH_NAME} ${DOCKERHUB_REPO_DEV}:latest"
                            sh "docker push ${DOCKERHUB_REPO_DEV}:latest"
                        } else if (env.BRANCH_NAME == 'main') {
                            sh "docker tag ${DOCKERHUB_REPO_DEV}:${env.BRANCH_NAME} ${DOCKERHUB_REPO_PROD}:latest"
                            sh "docker push ${DOCKERHUB_REPO_PROD}:latest"
                        }
                    }
                }
            }
        }
        stage('Deploy to Server') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@3.94.163.231 <<EOF
                        sudo docker pull ${DOCKERHUB_REPO_PROD}:latest
                        sudo docker stop react-app || true
                        sudo docker rm react-app || true
                        sudo docker run -d -p 80:80 --name react-app ${DOCKERHUB_REPO_PROD}:latest
                        EOF
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}