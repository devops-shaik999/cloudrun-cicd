pipeline {
    agent any 

    environment {
        PROJECT_ID = 'dev-uk-123'
        IMAGE_NAME = 'cloudrunlab'
        DOCKER_REPO = 'afroz2022'
        REGION = 'us-central1'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/devops-shaik999/cloudrun-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REPO}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-password',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_REPO}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                script {
                    withCredentials([file(
                        credentialsId: 'gcp-service-account',
                        variable: 'GOOGLE_APPLICATION_CREDENTIALS'
                    )]) {

                        sh """
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project ${PROJECT_ID}
                        gcloud auth configure-docker

                        gcloud run deploy ${IMAGE_NAME} \
                            --image docker.io/${DOCKER_REPO}/${IMAGE_NAME}:${BUILD_NUMBER} \
                            --platform managed \
                            --region ${REGION} \
                            --allow-unauthenticated \
                            --quiet
                        """
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Pipeline Failed. Check logs."
        }
    }
}