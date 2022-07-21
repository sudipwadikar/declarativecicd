def mvn
def buildInfo
pipeline {
  agent any
    tools {
      maven 'maven-3.8.6'
      jdk 'java-11'
    }
environment {
        AWS_ACCOUNT_ID="053334083296"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="docker-demo"
        IMAGE_TAG="v1"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"	
  stages {
	stage('Submit Stack') {
            steps {
            sh "aws cloudformation create-stack --stack-name ec2sg --template-body file://ec2sg.yaml --region 'us-east-1'"    
            //sh "aws cloudformation delete-stack --stack-name ec2sg --region 'us-east-1'"
              }
             }	  
    stage('Execute_Maven') {
	  steps {
	    script {
		  sh 'mvn clean package'
        }			                      
      }
    }	
    stage('Server'){
	  steps{
	  rtServer (
		id: "Artifactory",
		url: 'http://50.17.169.114:8082/artifactory',
		username: 'admin',
		password: 'Welcome1$',
		bypassProxy: true,
		timeout: 300		  
	  )
	}
    }
	  stage('Upload'){
		  steps{
		  rtUpload (
		  serverId:"Artifactory",
			  spec: '''{
			  "files":[
			  {
			  "pattern":"*.war",
			  "target": "helloworld-libs-snapshot"
			  }
			  ]
			  }''',
		  )
		  }
	  }
	  stage ('Publish Build Info') {
		  steps {
		  	rtPublishBuildInfo (
			 serverId:"Artifactory"
			)
		  }
	  }
	  
	  
    stage('Test_Maven') {
	    steps {
		  sh 'mvn test'			                     
	    }
	    post {
                always {
			 junit '**/target/surefire-reports/TEST-*.xml'
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
  stage('Building image') {
	  steps{
          sh 'docker build -t sudipwadikar/springtest:$BUILD_NUMBER .'
      }
  }	  
  stage('Docker Container'){
    steps{
      withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
	  sh 'docker login -u ${docker_user} -p ${docker_pass}'
	  sh 'docker run -d -p 8050:8050 --name SpringbootApp sudipwadikar/springtest:$BUILD_NUMBER'
	  sh "docker push sudipwadikar/springtest:$BUILD_NUMBER"	    
	  }
    }
  }
	  
  // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{
	      withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]){
		 sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
		//sh 'docker login -u ${docker_user} -p ${docker_pass}'       
		//sh "docker push sudipwadikar/springtest:$BUILD_NUMBER"	 
         }
        }
      }	 
	/*  stage('SSH to Server'){
	  steps{
		sshagent(['Instance']) {
			sh 'ssh -i /home/ubuntu/.ssh/teraform ec2-user@50.16.155.25'	
		}
               
	}
}*/	  
  }	  	  
}
