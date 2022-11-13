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
	
	
	stage('Docker Security Scans') {
	     steps {
        	   
		  // Docker Bench for Security
		  sh '''
		  
		  	#sudo apt-get install git -y
			
		  	git clone https://github.com/docker/docker-bench-security.git
		  	cd docker-bench-security
			
			rm Dockerfile
			cp ../vote/Dockerfile .
		  
		  	sudo sh docker-bench-security.sh > DSSReport
		  	#sh docker-bench-security.sh
			
			\n
			cat DSSReport
			\n
		
		   '''
		     
		     
		  // Dive tool
		  sh '''
		  	
			wget https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_linux_amd64.deb
			sudo apt install ./dive_0.9.2_linux_amd64.deb
			
			dive $docker_image > DiveReport
			
			\n
			cat DiveReport
			\n
			
		  '''
		     
		  
		  // Trivy tool
		  sh '''
		  
		  	sudo apt-get install wget apt-transport-https gnupg lsb-release
			wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
			echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
			sudo apt-get update
			sudo apt-get install trivy
			
			trivy image $docker_image > TrivyReport
			
			
			\n
			cat TrivyReport
			\n
			
			
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
		  
		  Docker Swarm Depolyment
		  
		  ----------------------------------------------------------"
		  
		  #docker swarm init --advertise-addr 172.31.90.195
		  
		  docker info
		  
		  docker service create --name vote --replicas=2 $docker_image
		  
		  '''
	      }
	}
        
    }
}
