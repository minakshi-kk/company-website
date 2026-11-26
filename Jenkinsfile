pipeline {
    agent none

    environment {
        DOCKER_HUB_USER = 'minakshi2004'
        DOCKER_IMAGE = 'company-website'
        DOCKER_TAG = 'latest'
        CONTAINER_NAME = 'company-website'
    }

    stages {

        stage('Checkout') {
            agent {
                label 'build-agent'
            }
            steps {
                git branch: 'main',
                    url: 'https://github.com/minakshi-kk/company-website.git'
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'build-agent'
            }
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Verify Image') {
            agent {
                label 'build-agent'
            }
            steps {
                sh '''
                    docker images
                    docker image inspect ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }

        stage('Push to Docker Hub') {
            agent {
                label 'build-agent'
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credential',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} \
                            $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker push $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            agent {
                label 'deploy-agent'
            }
            steps {
                sh '''
                    docker pull ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}

                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 9090:80 \
                        ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}



