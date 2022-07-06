def mvn
pipeline {
  agent any
    tools {
      maven 'maven-3.8.6'
    }
  stages {
   stage ('Maven Build') {
	   steps {
        sh "mvn clean package"
      }
    }
  stage('Build Docker Image'){
    steps{
      sh 'docker build -t dileep95/dileep-spring:$BUILD_NUMBER .'
    }
  }
  stage('Docker Container'){
    steps{
      withCredentials([usernameColonPassword(credentialsId: 'docker_dileep_creds', variable: 'DOCKER_PASS')]) {
      sh 'docker push dileep95/dileep-spring:$BUILD_NUMBER'
	  sh 'docker run -d -p 8050:8050 --name SpringbootApp dileep95/dileep-spring:$BUILD_NUMBER'
    }
    }
  }  

  }
}

