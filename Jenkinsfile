pipeline {
  agent {
    docker {
        image 'node20-docker'
        args '''
            --network aws-elastic-beanstalk-express-js-sample_jenkins_network
            --add-host docker:172.18.0.2
            -v /certs/client:/certs/client:ro
            -e DOCKER_HOST=tcp://docker:2376
            -e DOCKER_CERT_PATH=/certs/client
            -e DOCKER_TLS_VERIFY=1
        '''
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
        script {
            def hasTestScript = sh(
                script: "node -e \"const p=require('./package.json'); process.exit(p.scripts && p.scripts.test ? 0 : 1)\"",
                returnStatus: true
            )

            if (hasTestScript == 0) {
                sh 'npm test'
            } else {
                echo 'No test script found in package.json. Skipping unit tests.'
            }
        }
    }
}

        stage('Build Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    sh '''
                        docker version

                        docker build \
                            -t ${IMAGE_NAME}:${BUILD_NUMBER} .

                        docker tag \
                            ${IMAGE_NAME}:${BUILD_NUMBER} \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                    '''
                }
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

                        docker push \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker tag \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                        docker push \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

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
