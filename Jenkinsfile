pipeline {
    agent any

    tools {
        maven 'maven3'   
        jdk 'jdk7'     
    }

    environment {
        SONAR_TOKEN = credentials('new')  
    }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'new', variable: 'SONAR_TOKEN')]) {
                sh '''
       mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
        -Dsonar.projectKey=myapp \
        -Dsonar.projectName=myapp \
        -Dsonar.sources=src \
        -Dsonar.host.url=http://65.0.18.45:9000/ \
        -Dsonar.token=$SONAR_TOKEN
                              '''
                }
            }
        }

    }

    post {
        success {
            echo 'Build and SonarQube analysis completed successfully!'
        }
        failure {
            echo 'Build or SonarQube analysis failed.'
        }
    }
}
