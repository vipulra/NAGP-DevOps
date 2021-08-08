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
      stage('Create Docker Image') {
         steps {
            bat "docker build -t i-${username}-master --no-cache -f Dockerfile ."
         }
      }
      stage('Move our image to Docker registry') {
         steps {
            bat "docker tag i-${username}-master ${registry}:${BUILD_NUMBER}"
            withDockerRegistry([credentialsId: 'Test_Docker', url: ""]) {
               bat "docker push ${registry}:${BUILD_NUMBER}"
            }
         }
      }
      stage('Deploying our image') {
         steps {
            bat "docker run --name c-${username}-master -d -p 7100:8800 ${registry}:${BUILD_NUMBER}"
         }
      }

   }
}
