
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Jenkins credentials ID
        IMAGE_NAME = 'vineethakondepudi/hotstar'
        IMAGE_TAG = "${BUILD_NUMBER}"
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
            steps { sh """ docker rmi -f hotstar1 || true docker build -t hotstar1 . """ } } 
        
        stage('Deploy Container') { steps { sh """ docker rm -f hotstar || true docker run -d --name hotstar -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:${IMAGE_TAG} """ } }
        stage('Deploy to Nexus') { steps { withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jdk7', traceability: true) { sh 'mvn deploy' } } }
    }
}
