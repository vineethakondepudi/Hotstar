
pipeline {
    agent any
  tools {
        jdk 'jdk7'        
        maven 'maven3'      
    }
    environment {
        HOST_PORT = '8008'
        CONTAINER_PORT = '8080'
         SONARQUBE_ENV = 'sq'
    }

    stages {
        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps { sh """ docker rmi -f hotstar1 || true docker build -t hotstar1 . """ } } 
        
        stage('Deploy Container') { steps { sh """ docker rm -f hotstar || true docker run -d --name hotstar -p ${HOST_PORT}:${CONTAINER_PORT} hotstar1 """ } }
        stage('Deploy to Nexus') { 
            steps { withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jdk7', traceability: true) { sh 'mvn deploy' } } }
 stage('SonarQube Analysis') {
          stage('SonarQube Analysis') {
    steps {
        sh 'mvn clean verify sonar:sonar -Dsonar.host.url=http://65.0.18.45:9000 -Dsonar.login=squ_7566eb2cbfcff1ac2e0fbe98588f0a6324516cd1'
    }
}

        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
    }
}
