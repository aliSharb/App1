pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "alisharb/app1:latest"
        DEPLOYMENT_FILE = "deployment.yaml"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                   sh '''
            if [ -d "App1" ]; then
                rm -rf App1
            fi
            git clone https://github.com/aliSharb/App1
            '''
            
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE App1/"
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push $DOCKER_IMAGE"
                    }
                }
            }
        }

        stage('Clean Up Local Docker Image') {
            steps {
                script {
                    sh "docker rmi $DOCKER_IMAGE"
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                script {
                    sh "sed -i 's|image: .*|image: $DOCKER_IMAGE|' App1/$DEPLOYMENT_FILE"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f App1/$DEPLOYMENT_FILE"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution finished."
        }
        success {
            echo "Deployment was successful!"
        }
        failure {
            echo "Pipeline failed! Check logs."
        }
    }
}
