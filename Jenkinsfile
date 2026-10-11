
pipeline {
    agent any

    environment {
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/php-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Ritesh123-a/CI-CD-PHP-App-GitHub-Jenkins-Docker-Hub.git'
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                        -t $IMAGE_NAME:$IMAGE_TAG \
                        -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d \
                        --name test-$BUILD_NUMBER \
                        -p 9090:80 \
                        $IMAGE_NAME:$IMAGE_TAG

                    sleep 5

                    curl -f http://localhost:9090
                '''
            }

            post {
                always {
                    sh 'docker rm -f test-$BUILD_NUMBER || true'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" |
                            docker login -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE_NAME:$IMAGE_TAG
                        docker push $IMAGE_NAME:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f php-app || true

                    docker run -d \
                        --name php-app \
                        --restart unless-stopped \
                        -p 80:80 \
                        $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            sh 'docker image prune -f'
        }
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
