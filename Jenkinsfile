pipeline {
    agent any
 environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Jenkins credentials ID
        IMAGE_NAME = 'vineethakondepudi/hotstar'
        IMAGE_TAG = '${BUILD_NUMBER}'
    }
    stages {
              stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker rmi -f hotstar:v1 || true
            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }
         stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                        sh """
                          echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                          docker push ${IMAGE_NAME}:${IMAGE_TAG}
                          docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f con8 || true
                    docker run -d --name con8 -p 8008:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
        
    }
}
