pipeline {

    agent any

    environment {

        DOCKERHUB_USERNAME = "purushothdoc"

        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/healthcare-frontend"
        BACKEND_IMAGE  = "${DOCKERHUB_USERNAME}/healthcare-backend"
        DATABASE_IMAGE = "${DOCKERHUB_USERNAME}/healthcare-database"

        IMAGE_TAG = "${BUILD_NUMBER}"

        APP_SERVER = "YOUR_APPLICATION_SERVER_PUBLIC_IP"
    }

    stages {

        // =================================================
        // CHECKOUT
        // =================================================

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // =================================================
        // BUILD FRONTEND
        // =================================================

        stage('Build Frontend Image') {
            steps {
                sh """
                    docker build \
                      -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      ./frontend
                """
            }
        }

        // =================================================
        // BUILD BACKEND
        // =================================================

        stage('Build Backend Image') {
            steps {
                sh """
                    docker build \
                      -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                      ./backend
                """
            }
        }

        // =================================================
        // BUILD DATABASE
        // =================================================

        stage('Build Database Image') {
            steps {
                sh """
                    docker build \
                      -t ${DATABASE_IMAGE}:${IMAGE_TAG} \
                      ./database
                """
            }
        }

        // =================================================
        // PUSH TO DOCKER HUB
        // =================================================

        stage('Push Images to Docker Hub') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
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

        // =================================================
        // DEPLOY TO APPLICATION SERVER
        // =================================================

        stage('Deploy Application') {

            steps {

                sshagent(credentials: ['13.201.25.178']) {

                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} '
                            cd /opt/healthcare-app &&
                            export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME} &&
                            export IMAGE_TAG=${IMAGE_TAG} &&
                            docker compose pull &&
                            docker compose up -d &&
                            docker compose ps
                        '
                    """
                }
            }
        }

        // =================================================
        // VERIFY DEPLOYMENT
        // =================================================

        stage('Verify Deployment') {

            steps {

                sshagent(credentials: ['13.201.25.178']) {

                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} '
                            docker ps
                        '
                    """
                }
            }
        }
    }

    // =====================================================
    // POST ACTIONS
    // =====================================================

    post {

        success {
            echo "=============================================="
            echo "Healthcare application deployed successfully."
            echo "Jenkins Build Number: ${BUILD_NUMBER}"
            echo "Docker Image Tag: ${IMAGE_TAG}"
            echo "=============================================="
        }

        failure {
            echo "Healthcare application deployment failed."
        }
    }
}

