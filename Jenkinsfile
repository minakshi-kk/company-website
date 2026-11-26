pipeline {
    agent {
        label 'jenkins'
    }

    environment {
        DOCKER_IMAGE = 'company-website'
        DOCKER_TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/minakshi-kk/company-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t company-website:latest .
                '''
            }
        }

        stage('Verify Image') {
            steps {
                sh '''
                    docker images
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credential',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag company-website:latest \
                            "DOCKER_USER/company-website:latest"

                        docker push $DOCKER_USER/company-website:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            agent {
                label 'build '
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
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker pull "$DOCKER_USER/comapany-website:latest"

                        docker stop company-website || true
                        docker rm company-website || true

                        docker run -d \
                            --name company-website \
                            -p 9090:80 \
                            $DOCKER_USER/company-website:latest"

                        docker logout
                    '''
                }
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
    }
}
