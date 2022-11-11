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
	    	docker_image = "498747127127.dkr.ecr.us-east-1.amazonaws.com/dockerimages:latest"
	    	
		//task_def_arn = "arn:aws:ecs:us-east-1:407730735276:task-definition/first-run-task-definition:10"
        	//cluster = "CICD"
        	//exec_role_arn = "arn:aws:iam::407730735276:role/ecsTaskExecutionRole"
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
       
	    
	stage('Dockerize') {
	     steps {
        	   // Get SHA1 of current commit
		   script {
            		   commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        	   }
                  // Build the Docker image
                  sh '''
		                
		  	cd vote/
			      
			#aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 498747127127.dkr.ecr.us-east-1.amazonaws.com
		
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
	
	
	stage('Tests') {
	     steps {
                  sh '''
		  
		  echo "Functional Tests"
		  
		  '''
	      }
	}
	
	stage('Depolyment') {
	     steps {
                  sh '''
		  
		  echo "----------------------------------------------------
		  /n
		  Docker Swarm Depolyment
		  /n----------------------------------------------------------"
		  
		  docker swarm init --advertise-addr 172.31.90.195
		  
		  docker info
		  
		  docker service create --name vote --replicas=2 $docker_image
		  
		  '''
	      }
	}
        
    }
}
