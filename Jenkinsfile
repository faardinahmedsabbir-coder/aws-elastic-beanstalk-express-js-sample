```groovy
pipeline {
    agent {
        docker {
            image 'node:16'
        }
    }

    environment {
        IMAGE_NAME = 'aws-node-app'
        DOCKER_REGISTRY = 'docker.io'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                            -u "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs for details.'
        }

        always {
            echo "Build number: ${BUILD_NUMBER}"
        }
    }
}
```
