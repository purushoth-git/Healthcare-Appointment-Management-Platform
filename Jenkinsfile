pipeline {

    agent any

    environment {

        DOCKERHUB_USERNAME = credentials('dockerhub-credentials')

        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/healthcare-frontend"
        BACKEND_IMAGE   = "${DOCKERHUB_USERNAME}/healthcare-backend"
        DATABASE_IMAGE  = "${DOCKERHUB_USERNAME}/healthcare-database"

        IMAGE_TAG = "${BUILD_NUMBER}"

        APP_SERVER = "YOUR_APP_SERVER_PUBLIC_IP"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git'
            }
        }

        stage('Build Frontend') {
            steps {
                sh """
                    docker build \
                      -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      ./frontend
                """
            }
        }

        stage('Build Backend') {
            steps {
                sh """
                    docker build \
                      -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                      ./backend
                """
            }
        }

        stage('Build Database') {
            steps {
                sh """
                    docker build \
                      -t ${DATABASE_IMAGE}:${IMAGE_TAG} \
                      ./database
                """
            }
        }

        stage('Push Images') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push "$FRONTEND_IMAGE:$IMAGE_TAG"
                        docker push "$BACKEND_IMAGE:$IMAGE_TAG"
                        docker push "$DATABASE_IMAGE:$IMAGE_TAG"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {

            steps {

                sshagent(credentials: ['app-server-ssh']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER "
                            cd /opt/healthcare-app &&
                            export DOCKERHUB_USERNAME=$DOCKERHUB_USERNAME &&
                            export IMAGE_TAG=$IMAGE_TAG &&
                            docker compose pull &&
                            docker compose up -d &&
                            docker compose ps
                        "
                    }
                }
            }
        }

        stage('Verify Deployment') {

            steps {

                sshagent(credentials: ['app-server-ssh']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER "
                            docker ps
                        "
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "Healthcare application deployed successfully."
            echo "Docker image tag: ${IMAGE_TAG}"
        }

        failure {
            echo "Healthcare application deployment failed."
        }
    }
}
