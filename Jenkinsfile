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
  stages {
    stage('Artifactory_Configuration') {
      steps {
        script {
		  rtMaven.tool = 'maven-3.8.6'
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
		  sh 'mvn clean package'
        }			                      
      }
    }	
    stage('SonarQube_Analysis'){
            steps{
                script{
                withSonarQubeEnv(installationName: 'sonar-9.5', credentialsId: 'Jenkins-sonar-token'){
                sh 'mvn sonar:sonar'        
                }
                }
            }
        }
	stage('Quality_Gate') {
	  steps {
	    timeout(time: 2, unit: 'MINUTES') {
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
      sh 'docker build -t sudipwadikar/springtest:$BUILD_NUMBER .'
    }
  }	  	 
  stage('Docker Container'){
    steps{
      withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
	  sh 'docker login -u ${docker_user} -p ${docker_pass}'
	  sh 'docker run -d -p 8050:8050 --name SpringbootApp dileep95/springtest:$BUILD_NUMBER'
	  }
    }
  }
}	  	  
}
