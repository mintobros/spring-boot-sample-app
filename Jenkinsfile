pipeline {
  agent any
  tools { 
      maven 'M2_HOME' 
      jdk 'JAVA_HOME'
  }
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker_hub_credentials')
    DEV_EC2_SERVER = '13.234.67.114'
    DEV_EC2_USER = 'ec2-user'            
  }

  // Fetch code from GitHub

  stages {
    stage ('Initialize') {
      steps {
          sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
          ''' 
      }
    }
    stage('Clone SoureCode') {
      steps {
        git branch: 'main', url: 'https://github.com/mintobros/spring-boot-sample-app-main-.git'

      }
    }

   // Build Java application

    stage('Build') {
      steps {
        sh 'mvn clean install'
      }

     // Post building archive Java application

      post {
        success {
          archiveArtifacts artifacts: '**/target/*.jar'
        }
      }
    }

  // Test Java application

    stage('Unit Test') {
      steps {
        sh 'mvn test'
      }
    }
    //sonarque server
    stage("SonarQube analysis") {
	    steps {
		  
		sh 'mvn sonar:sonar -Dsonar.host.url=http://13.234.67.114:9000/ -Dsonar.login=squ_4436e624a007192bf52ab2cf614adea153e8af13'
		       
	      }
      }

   // Build docker image in Jenkins

    stage('Prepare Image') {

      steps {
        sh 'docker build -t spring-boot-sample-app:latest .'
        sh 'docker tag spring-boot-sample-app nehal9984/spring-boot-sample-app:latest'
      
      }
    }

   // Login to DockerHub before pushing docker Image

    stage('Login to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u    $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }

   // Push image to DockerHub registry

    stage('Push Image to dockerHUb') {
      steps {
        sh 'docker push nehal9984/spring-boot-sample-app:latest'
      }
      post {
        always {
          sh 'docker logout'
        }
      }

    }

   // Pull docker image from DockerHub and run in EC2 instance (dev environment) 

    stage('Dev Environment') {
      steps {
        script {
          sshagent(credentials: ['awscred']) {
          sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker stop spring-boot-sample-app-dev || true && docker rm spring-boot-sample-app-dev || true'"
      sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker pull nehal9984/spring-boot-sample-app'"
          sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker run --name spring-boot-sample-app-dev -d -p 8081:8081 nehal9984/spring-boot-sample-app'"
          }
        }
      }
    }
    // Pull docker image from DockerHub and run in stage EC2 instance (test environment)
      stage('Test Environment') {
        
      steps {
       script {
               def userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
               echo 'userInput: ' + userInput
               if(userInput == true) {
                    sshagent(credentials: ['awscred']) {
                      sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker stop spring-boot-sample-app-test || true && docker rm spring-boot-sample-app-test || true'"
                      sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker pull nehal9984/spring-boot-sample-app'"
                      sh "ssh -o StrictHostKeyChecking=no ${DEV_EC2_USER}@${DEV_EC2_SERVER} 'docker run --name spring-boot-sample-app-test -d -p 8082:8081 nehal9984/spring-boot-sample-app'"
                    }
                } else {
                    // not do action
                 echo "Action was aborted."
                }
          
        }
      }
    }
  }
}
