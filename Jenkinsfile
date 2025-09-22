pipeline {
    agent any
 environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Jenkins credentials ID
        IMAGE_NAME = 'vineethakondepudi/hotstar'
        IMAGE_TAG = '${BUILD_NUMBER}'
     HOST_PORT = '8008'
    CONTAINER_PORT = '8080'
    }
    stages {
              stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

      stage('Build Docker Image') {
    steps {
        sh """
            docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        """
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
        sh """
            docker rm -f hotstar || true
            docker run -d --name hotstar -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:${IMAGE_TAG}
        """
    }
}
   stage('Deploy Container') {
    steps {
        sh """
           withMaven(globalMavenSettingsConfig: 'settings.xml', maven: 'manen3', traceability: true) {
    mvn deploy
}
        """
    }
}
        
    }
}
