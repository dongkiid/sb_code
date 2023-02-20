pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }
  environment {
    gitName = 'dongkiid'
    gitEmail = 'sieuri@naver.com'
    githubCredential = 'git_cre'
    dockerHubRegistry = 'dongkiid/sbimage'
    dockerHubRegistryCredential = 'docker_cre'
  }
  
  stages {
    stage('Checkout Github') {
      steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: githubCredential, url: 'https://github.com/dongkiid/sb_code.git']]])
        }
      post {
        failure {
          echo 'Repository clone failure'
        }
	success {
	 echo 'Repository clone success'
        }
      }
    }


    stage('Maven Build') {
      steps {
         sh 'mvn clean install'
         }
      post {
        failure {
          echo 'Maven jar build failure'
        }
        success {
          echo 'Maven jar build success'  
        }
      }
    }
    stage('Docker Image Build') {
     steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
      sh "docker build -t ${dockerHubRegistry}:latest ."
      }
    post {
     failure {
       echo 'Docker image build failure'
     }
     success {
      echo 'Docker image build success'  
     }
    }
   }
   stage('Docker Image Push') {
     steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
          }  
      }
      post {
       failure {
         echo 'Docker Image Push failure'
         sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
         sh "docker rmi ${dockerHubRegistry}:latest"
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker rmi ${dockerHubRegistry}:latest"
          slackSend (color: '#0AC9FF', message: "SUCCESS: Docker Image Push '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }
    stage('Docker Container Deploy') {
      steps {
          sh "docker rm -f spring"
          sh "docker run -dp 7979:8085 --name spring ${dockerHubRegistry}:${currentBuild.number}"
          }
      post {
        failure {
          echo 'Container Deploy failure'
        }
        success {
          echo 'Container Deploy success'  
        }
      }
    }
  }
}


