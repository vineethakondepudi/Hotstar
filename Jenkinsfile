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

        
    }
}
