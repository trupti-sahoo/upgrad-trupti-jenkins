pipeline {
  agent {
    label 'worker'
  }
   
  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/trupti-sahoo/upgrad-trupti-jenkins']]])
      }
    }
    
     stage('Stopping the container') {
      steps {
        script {
          sh 'docker kill $(docker ps -q)'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {     
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 753280441213.dkr.ecr.us-east-1.amazonaws.com
          docker build -t jenkins:${BUILD_NUMBER} .
          docker tag jenkins:${BUILD_NUMBER} 753280441213.dkr.ecr.us-east-1.amazonaws.com/jenkins:${BUILD_NUMBER}
          docker push 753280441213.dkr.ecr.us-east-1.amazonaws.com/jenkins:${BUILD_NUMBER} 
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 753280441213.dkr.ecr.us-east-1.amazonaws.com/jenkins:${BUILD_NUMBER}'
          sh 'docker rmi jenkins:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 753280441213.dkr.ecr.us-east-1.amazonaws.com
          docker pull 753280441213.dkr.ecr.us-east-1.amazonaws.com/jenkins:${BUILD_NUMBER}
          docker run -d -p 8080:8081 753280441213.dkr.ecr.us-east-1.amazonaws.com/jenkins:${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
