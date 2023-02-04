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
	    	
		// ECS environment variables
	    	
		//task_def_arn = "arn:aws:ecs:us-east-1:407730735276:task-definition/first-run-task-definition:10"
	    	task_def_arn = "arn:aws:ecs:us-east-1:498747127127:task-definition/vote:2"
        	cluster = "CICD_ECS"
        	exec_role_arn = "arn:aws:iam::498747127127:role/ecsTaskExecutionRole"
    }
    
    stages{
        
       stage('GetCode'){
            steps{
                git branch: 'cicd_ecs', url: 'https://github.com/sagarainapur/POC-for-MSCS.git'
            }
       }
       
	
       stage('SonarQube analysis') {
            //def scannerHome = tool 'SonarScanner 4.0';
            steps{
                withSonarQubeEnv('SonarQube-9.7.1') { 
                // If you have configured more than one global server connection, you can specify its name
                //sh "${scannerHome}/bin/sonar-scanner"
                //sh "mvn clean verify sonar:sonar -Dsonar.projectKey=demo-app -Dsonar.host.url=http://107.22.241.117:9000 -Dsonar.login=sqp_e2ba3b07b8fe290f9235c068c949e58b18f5d0e6"
                
		sh "mvn clean verify sonar:sonar -Dsonar.projectKey=CICD -Dsonar.host.url=http://52.5.131.155:9000 -Dsonar.login=sqp_6c018a946c0729440463adbe15bd5b2bcd719365"
		
		
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
		        ls -lash
			
		  	cd vote/
			
			ls -lash
			
			chmod +x Dockerfile app.py requirements.txt
			
			ls -lash
			##cd /var/lib/jenkins/workspace/CICD_ECS@tmp/durable-280cb0c1/
			
			#chmod +x script.sh
			
			# Build the Docker image
        		#docker build -t ${docker_repo_uri}:${commit_id} .
			docker build -t ${docker_repo_uri}:latest .
						
        		# Get Docker login credentials for ECR
        		aws ecr get-login --no-include-email --region ${region} | sh
			
        		# Push Docker image
        		# docker push ${docker_repo_uri}:${commit_id}
			docker push ${docker_repo_uri}:latest
			
        		# Clean up
        		# docker rmi -f ${docker_repo_uri}:${commit_id}
			docker rmi -f ${docker_repo_uri}:latest
			
			
			
		
		   ''' 
	      }
	}
	
		
		
	stage('Trivy Scans') {
	     steps {
		     
		  
		  // Trivy tool
		  sh '''
		  
		  	echo "Trivy tool scan"
			
		  	sudo apt-get install wget apt-transport-https gnupg lsb-release -y
			wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
			echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
			sudo apt-get update -y
			sudo apt-get install trivy -y
						
			trivy image $docker_image > TrivyReport
			
			
			\n
			cat TrivyReport
			\n
			
			
		  '''
		  
	      }
	}
	
	
	
	stage('ECS Depolyment') {
		
	     steps {

		// Override image field in taskdef file - taskdef.json
        	sh "sed -i 's|{{image}}|${docker_repo_uri}:latest|' taskdef.json"
        	// Create a new task definition revision
        	sh "aws ecs register-task-definition --execution-role-arn ${exec_role_arn} --cli-input-json file://taskdef.json --region ${region}"
        	// Update service on Fargate
        	sh "aws ecs update-service --cluster ${cluster} --service vote --task-definition ${task_def_arn} --region ${region}"
	      }
	}
        
    }
}
