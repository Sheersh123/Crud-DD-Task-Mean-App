pipeline {
    agent any

    environment {
        // Image names on Docker Hub (change to your Docker Hub username)
        DOCKERHUB_USER = "sheershsinha"
        BACKEND_IMAGE  = "${DOCKERHUB_USER}/mean-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/mean-frontend"
    }

    triggers {
        // Optional: build on every push
        pollSCM('H/5 * * * *')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                // Uses Token-01 for GitHub (if needed in Jenkins global config)
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    dir('backend') {
                        sh 'ls -la'
                    }
                }
                withCredentials([usernamePassword(credentialsId: 'Token-02',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        echo "Building backend image..."
                        docker build -t ${BACKEND_IMAGE}:latest backend

                        echo "Pushing backend image..."
                        docker push ${BACKEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    dir('frontend') {
                        sh 'ls -la'
                    }
                }
                withCredentials([usernamePassword(credentialsId: 'Token-02',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        echo "Building frontend image..."
                        docker build -t ${FRONTEND_IMAGE}:latest frontend

                        echo "Pushing frontend image..."
                        docker push ${FRONTEND_IMAGE}:latest
                    """
                }
            }
        }

        // Optional: basic sanity check using docker-compose (CI smoke test)
        stage('Smoke Test with Docker Compose') {
            steps {
                sh """
                    echo "Starting services with docker-compose (using local images)..."
                    docker compose down || true
                    docker compose up -d

                    echo "Waiting a bit for services to be ready..."
                    sleep 20

                    echo "Checking backend health..."
                    curl -f http://localhost:8080/ || exit 1

                    echo "Checking frontend (HTTP 200 expected)..."
                    curl -f http://localhost:8081/ || exit 1

                    echo "Stopping services..."
                    docker compose down
                """
            }
        }

        // Optional: deploy to remote EC2 via SSH (if you configure it)
        // stage('Deploy to EC2') {
        //     steps {
        //         withCredentials([
        //             sshUserPrivateKey(credentialsId: 'EC2-SSH-KEY',
        //                               keyFileVariable: 'SSH_KEY',
        //                               usernameVariable: 'EC2_USER'),
        //             string(credentialsId: 'EC2-HOST', variable: 'EC2_HOST')
        //         ]) {
        //             sh """
        //                 ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" $EC2_USER@$EC2_HOST '
        //                     cd /home/ubuntu/Crud-DD-Task-Mean-App &&
        //                     docker compose pull &&
        //                     docker compose up -d
        //                 '
        //             """
        //         }
        //     }
        // }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Build & push of frontend and backend images completed successfully."
        }
        failure {
            echo "Build failed. Check console output for details."
        }
    }
}
