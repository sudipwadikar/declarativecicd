def mvn
def server = Artifactory.server 'artifactory'
def rtMaven = Artifactory.newMavenBuild()
def buildInfo
pipeline {
  agent any
    tools {
      maven 'maven-3.8.6'
      jdk 'java-11'
    }
  environment {
    SONAR_HOME = "${tool name: 'sonar-9.5', type: 'hudson.plugins.sonar.SonarRunnerInstallation'}"
  }  
  stages {
    stage('Artifactory_Configuration') {
      steps {
        script {
		  rtMaven.tool = 'Maven'
		  rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
		  buildInfo = Artifactory.newBuildInfo()
		  rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot', server: server
          buildInfo.env.capture = true
        }			                      
      }
    }
    stage('Execute_Maven') {
	  steps {
	    script {
		  rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
        }			                      
      }
    }	
    stage('SonarQube_Analysis') {
      steps {
	    script {
          scannerHome = tool 'sonar-scanner'
        }
        withSonarQubeEnv('sonar') {
      	  sh """${scannerHome}/bin/sonar-scanner"""
        }
      }	
    }	
	stage('Quality_Gate') {
	  steps {
	    timeout(time: 1, unit: 'MINUTES') {
		  waitForQualityGate abortPipeline: true
        }
      }
    }
   stage('Deleting docker images and Containers'){
    steps{
     sh 'chmod +x delete_cont.sh'
     sh './delete_cont.sh'	      
    }
  }
  stage('Build Docker Image'){
    steps{
      sh 'docker build -t dileep95/springtest:$BUILD_NUMBER .'
    }
  }	  	 
  stage('Docker Container'){
    steps{
      withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
	  sh 'docker login -u ${docker_user} -p ${docker_pass}'
      	  sh 'docker push dileep95/springtest:$BUILD_NUMBER'
	  sh 'docker run -d -p 8050:8050 --name SpringbootApp dileep95/springtest:$BUILD_NUMBER'
	  }
    }
  }
}	  	  
}
