pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk7'
    }

    stages {
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build image'){
            steps{
            sh """
            docker rmi -f hotstar1 || true
                docker build -t hotstar1 .
                    """
            }
        }
     stage('Docker run container'){
         steps{
         sh """
         docker rm -f jenkins || true
         docker run -itd --name hotstar -p 8001:8080 hotstar1
         """
         }
     }

       stage('SonarQube Analysis') {
    steps {
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
            echo 'Build and SonarQube analysis completed successfully!'
        }
        failure {
            echo 'Build or SonarQube analysis failed.'
        }
    }
}
