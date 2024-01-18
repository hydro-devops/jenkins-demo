pipeline {
  environment {
    dockerimagename = "hoaithuongdata/test-app"
    dockerImage = ""
    
    urlK8s = "https://5ede-1-55-81-251.ngrok-free.app/" // for test remote k8s cluster
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git 'https://github.com/hydro-devops/jenkins-demo.git'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        script {
          def scannerHome = tool 'SonarQube Scanner';
          withSonarQubeEnv(installationName= 'sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image') {
      environment {
          registryCredential = 'docker-hub'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploy App on k8s') {
      steps {
        withCredentials([
            string(credentialsId: 'my_kubernetes_serect', variable: 'api_token')
            ]) {
                          
             sh 'kubectl --token $api_token --server  ${urlK8s} --insecure-skip-tls-verify=true delete  deployment --ignore-not-found=true test-app'
             sh 'kubectl --token $api_token --server  ${urlK8s} --insecure-skip-tls-verify=true apply -f deployment.yaml'
             sh 'kubectl --token $api_token --server ${urlK8s} --insecure-skip-tls-verify=true apply -f service.yaml'
               }
            }
        }
  }
}
