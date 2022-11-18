pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('GIT Checkout') {
            steps {
            git 'https://github.com/KP-Devops/k8sdemo.git'
            }
          }
          
        stage('Maven Build') {
            steps {
            sh 'mvn clean install'
            }
        }
        
        stage('ExecuteSonarQubeReport'){
		    steps{
	    	  withSonarQubeEnv('sonarqube8.9'){
			    sh 'mvn sonar:sonar'
		       }   
		    }
	    }
        
        stage('NexusArtifactUploader'){
            steps{
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'springbootApp', 
                        classifier: '', 
                        file: '/var/lib/jenkins/.m2/repository/com/tcs/angularjs/springbootApp/1.0.0/springbootApp-1.0.0.jar', 
                        type: 'jar'
                        ]
                    ], 
                    credentialsId: 'NEXUS', 
                    groupId: 'com.tcs.angularjs', 
                    nexusUrl: '13.232.235.41:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'sprintboot-release', 
                    version: '1.0.0'
            }
        }
        
        stage('Docker Build, Image List and Tag'){
            steps{
                sh 'docker build -t kpdocker .'
                sh 'docker image list '
                sh 'docker tag kpdocker:latest 296475210819.dkr.ecr.ap-south-1.amazonaws.com/kpdocker:latest'
            }
        }
            
        stage('AWS Login'){
            steps{
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 296475210819.dkr.ecr.ap-south-1.amazonaws.com'
            }
        }
    
        stage('DockerPush to AWS ECR'){
            steps{
                sh 'docker push 296475210819.dkr.ecr.ap-south-1.amazonaws.com/kpdocker:latest'
            }
        }   

        stage('EKS Deployment'){
            steps{
                script{
                    withKubeConfig([credentialsId: 'K8', serverUrl: '']) 
                    {
                    sh ('kubectl apply -f  deployment.yml')
                    }
                  }
                }    
              }  
    }
}