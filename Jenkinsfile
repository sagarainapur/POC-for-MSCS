pipeline{
    agent any
    tools {
    maven 'maven'
    }
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
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
//    def scannerHome = tool 'SonarScanner 4.0';
        steps{
        withSonarQubeEnv('SonarQube-9.7.1') { 
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=demo-app -Dsonar.host.url=http://107.22.241.117:9000 -Dsonar.login=sqp_e2ba3b07b8fe290f9235c068c949e58b18f5d0e6"
    }
        }
        }
       
    }
}
