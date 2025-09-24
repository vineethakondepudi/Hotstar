pipeline {
    agent any

    tools {
        jdk 'jdk7'        
        maven 'maven3'      
    }

    environment {
        HOST_PORT = '8008'
        CONTAINER_PORT = '8080'
        SONARQUBE_ENV = 'sq' // Make sure this matches the name in Jenkins SonarQube installations
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
                    docker rmi -f hotstar1 || true
                    docker build -t hotstar1 .
                """
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker rm -f hotstar || true
                    docker run -d --name hotstar -p ${HOST_PORT}:${CONTAINER_PORT} hotstar1
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
                withSonarQubeEnv("${SON
