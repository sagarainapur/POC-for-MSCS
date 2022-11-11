pipeline{
    agent any
    
    tools {
    maven 'maven'
    }
    
    // This section contains environment variables which are available for use in the pipeline's stages.
    environment {
		PATH = "$PATH:/usr/share/maven/bin"
        	region = "us-east-1"
        	docker_repo_uri = "498747127127.dkr.ecr.us-east-1.amazonaws.com/dockerimages"
		      task_def_arn = "arn:aws:ecs:us-east-1:407730735276:task-definition/first-run-task-definition:10"
        	cluster = "CICD"
        	exec_role_arn = "arn:aws:iam::407730735276:role/ecsTaskExecutionRole"
    }
    
    stages{
        
       stage('GetCode'){
            steps{
                git branch: 'main', url: 'https://github.com/sagarainapur/POC-for-MSCS.git'
            }
       }
        
       stage('Build'){
            steps{
                sh 'mvn clean package'
            }
       }
        
       stage('SonarQube analysis') {
            //def scannerHome = tool 'SonarScanner 4.0';
            steps{
                withSonarQubeEnv('SonarQube-9.7.1') { 
                // If you have configured more than one global server connection, you can specify its name
                //sh "${scannerHome}/bin/sonar-scanner"
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=demo-app -Dsonar.host.url=http://107.22.241.117:9000 -Dsonar.login=sqp_e2ba3b07b8fe290f9235c068c949e58b18f5d0e6"
                }
            }
        }
       
	    
	      stage('Dockerize1') {
	          steps {
        	   // Get SHA1 of current commit
		              script {
            		          commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        	        }
                  // Build the Docker image
                  sh '''
		                
		              cd vote
		
		              docker build -t ${docker_repo_uri}:latest .
		
                  # Get Docker login credentials for ECR
                  aws ecr get-login --no-include-email --region ${region} | sh
		
                  # Push Docker image
                  docker push ${docker_repo_uri}:latest
		
                  #Clean up
                  #docker rmi -f ${docker_repo_uri}:latest
		
		              '''
    	      }
	      }    
	    
	    
	    
        stage('Dockerizeold') {
	          steps {
        	   // Get SHA1 of current commit
		              script {
            		          commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        	        }
                   //Build the Docker image
                  sh "docker build -t ${docker_repo_uri}:${commit_id} ."
                // Get Docker login credentials for ECR
                sh "aws ecr get-login --no-include-email --region ${region} | sh"
                // Push Docker image
                sh "docker push ${docker_repo_uri}:${commit_id}"
                // Clean up
                sh "docker rmi -f ${docker_repo_uri}:${commit_id}"
    	    }
	}
        
    }
}
