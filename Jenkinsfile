pipeline {
    agent {
        label 'build'
    }

    environment {
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/${DOCKERHUB_REPOSITORY}"
        CONTAINER_NAME = "company-website"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPOSITORY_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Deploy to Server') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'deployment-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        -i "$SSH_KEY" \
                        "$SSH_USER@${DEPLOYMENT_SERVER_IP}" '

                            docker pull ${DOCKER_IMAGE}:latest

                            docker stop company-website || true
                            docker rm company-website || true

                            docker run -d \
                                --name company-website \
                                -p 9090:80 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:latest
                        '
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://${DEPLOYMENT_SERVER_IP}:9090
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins logs.'
        }
    }
}
