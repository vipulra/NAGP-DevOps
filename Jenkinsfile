pipeline {
   environment {
      username = 'vipulrastogi'
      registry = 'vipul7/nagp_devops'
   }
   agent any

   tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven"
   }

   stages {
      stage('Build') {
         steps {
            // Get some code from a GitHub repository
            git credentialsId: 'github-java-app', url: 'https://github.com/vipulra/NAGP-DevOps.git'

            // To run Maven on a Windows agent, use
            bat "mvn clean install"
         }

      }

      stage('Unit Testing') {
         steps {
            // Run unit test case
            bat "mvn test"
         }
      }

      // Code Analysis
      stage('Sonar Analysis') {
         environment {
            scannerHome = tool name: 'SonarQubeScanner'
         }
         steps {
            withSonarQubeEnv("Test_Sonar") {
               bat "mvn clean package sonar:sonar"
//                bat "mvn sonar:sonar"
            }
         }
      }
      
      stage('Docker Image') {
         steps {
            bat "docker build -t i-${username}-master:${BUILD_NUMBER} --no-cache -f Dockerfile ."
         }
      }
      
      stage('Container') {
         parallel {
          stage('PreContainer check') {
             steps{
		     script {
                	try {
                   		bat "docker rm -f c-${username}-master"
                	}
                	catch (Exception err) {
                     	//container is not present
                	}
		     }
              }
          }
          stage('Push image to docker hub') {
              steps {
               bat "docker tag i-${username}-master:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
		         bat "docker tag i-${username}-master:${BUILD_NUMBER} ${registry}:latest"
		         withDockerRegistry([credentialsId: 'Test_Docker', url:""]){
			         bat "docker push ${registry}:${BUILD_NUMBER}"
			         bat "docker push ${registry}:latest"     
		       }
           }
         }
         }
      }
      stage('Deploying our image') {
         steps {
            bat "docker run --name c-${username}-master -d -p 7100:8080 ${registry}:latest"
         }
      }
      
      stage ("Deploy to GKE"){
		steps{	
			bat "kubectl create -f deployment.yml -v=8"
// 	             bat "kubectl apply -f deployment.yaml"
		}
	}

   }
}
