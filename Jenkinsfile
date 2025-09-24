pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk7'
    }
    environment{
    image_name: 'hotstar1'
    image_tag: "${BUILD_NUMBER}"
    host_num: '8001'
    container_num: '8080'           
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
            docker rmi -f ${image_name}:${image_tag} || true
                docker build -t ${image_name}:${image_tag} .
                    """
            }
        }
     stage('Docker run container'){
         steps{
         sh """
         docker rm -f hotstar || true
         docker run -itd --name hotstar -p ${host_num}:${container_num} ${image_name}:${image_tag}
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
