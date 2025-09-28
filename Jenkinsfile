pipeline {
    agent any

   tools {
       maven 'maven3' 
       jdk 'jdk7' 
   }

    environment {
        IMAGE_NAME = 'vineethakondepudi/hotstar1'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = 'docker_credential'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📂 Checking out source code..."
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                echo "⚡ Building Maven project..."
                sh 'mvn clean package -DskipTests'
            }
        }

     stage('SonarQube Analysis') {
         steps { 
             echo "Running SonarQube analysis..." 
             withCredentials([string(credentialsId: 'new', variable: 'SONAR_TOKEN')]) {
                 sh ''' mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \ -Dsonar.projectKey=myapp \ -Dsonar.projectName=myapp \ -Dsonar.sources=src/main/java \ -Dsonar.tests=src/test/java \ -Dsonar.host.url=http://13.232.145.228:9000/ \ -Dsonar.token=$SONAR_TOKEN ''' 
             }
         }
     }
    }

   stage('Deploy to Nexus') { 
       steps {
           withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jdk7', traceability: true) {
               sh 'mvn deploy'
           }
       }
   }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                        sh """
                        echo "🐳 Building Docker image..."
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

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
                echo "🚀 Deploying to Kubernetes..."
                sh """
                sed -i 's|image: vineethakondepudi/hotstar1:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml
                kubectl apply -f k8s/deployment.yaml
                kubectl rollout status deployment/hotstar-deployment
                kubectl get pods -o wide
                """
            }
        }
    }

    post {
        success {
            echo '✅ Build, SonarQube, Nexus, Docker, and Kubernetes deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
