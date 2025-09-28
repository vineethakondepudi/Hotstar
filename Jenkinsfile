pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk7'
    }

    environment {
        IMAGE_NAME = 'vineethakondepudi/hotstar1'
        IMAGE_TAG = "${BUILD_NUMBER}"
        HOST_PORT = '8001'
        CONTAINER_PORT = '8080'
        CONTAINER_NAME = 'hotstar'
        DOCKERHUB_CREDENTIALS = 'docker_credential'   
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

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

     stage('Deploy to Kubernetes') {
    steps {
        echo "Deploying ${IMAGE_NAME}:${IMAGE_TAG} to Kubernetes..."
        sh """
        # Update image tag in deployment.yaml
        sed -i 's|image: vineethakondepudi/hotstar1:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml

        # Apply Kubernetes manifests
        kubectl apply -f k8s/deployment.yaml

        # Optional: check rollout status
        kubectl rollout status deployment/hotstar-deployment
        kubectl get pods -o wide
        """
    }
}

        
        stage('Deploy to Nexus') { 
            steps {
                withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jdk7', traceability: true) {
                    sh 'mvn deploy' 
                }
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
                    -Dsonar.host.url=http://3.110.210.127:9000/ \
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
