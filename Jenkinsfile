pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk7'
    }

    environment {
        IMAGE_NAME = 'hotstar'
        IMAGE_TAG = "${BUILD_NUMBER}"
        HOST_PORT = '8001'
        CONTAINER_PORT = '8080'
        DOCKER_SOCK = '/var/run/docker.sock'
    }

    stages {

        stage('Maven Build') {
            steps {
                echo "Building Maven project..."
                sh 'mvn clean package'
            }
        }

        stage('Docker Build Image') {
            steps {
                echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh """
                # Remove old image if exists
                docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true

                # Build new image
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Run Container') {
            steps {
                echo "Running container ${IMAGE_NAME}..."
                sh """
                # Remove old container if exists
                docker rm -f ${IMAGE_NAME} || true

                # Run new container
                docker run -d --name ${IMAGE_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:${IMAGE_TAG}

                # Verify container is running
                docker ps -a
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                withCredentials([string(credentialsId: 'new', variable: 'SONAR_TOKEN')]) {
                    sh '''
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
                    -Dsonar.projectKey=myapp \
                    -Dsonar.projectName=myapp \
                    -Dsonar.sources=src/main/java \
                    -Dsonar.tests=src/test/java \
                    -Dsonar.host.url=http://3.110.162.202:9000/ \
                    -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, Docker, and SonarQube stages completed successfully!'
        }
        failure {
            echo '❌ Build failed. Check logs for details.'
        }
    }
}
