pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubuser/php-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_CREDS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<user>/php-app.git'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG -t $IMAGE_NAME:latest .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                docker run -d --name test-$BUILD_NUMBER -p 9090:80 $IMAGE_NAME:$IMAGE_TAG
                sleep 5
                curl -f http://localhost:9090 || exit 1
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
                sh '''
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                docker push $IMAGE_NAME:$IMAGE_TAG
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker rm -f php-app || true
                docker run -d --name php-app -p 80:80 $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            sh 'docker image prune -f'
        }
    }
}