pipeline {
  environment {
    dockerimagename = "namdh1985/react-app"
    dockerImage = ""
    GIT_CREDENTIALS_ID = 'github-credentials'
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        script {
                    def gitRepo = 'https://github.com/namdh1985/jenkins-kubernetes-deployment.git'
                    def gitBranch = 'main'  
                    checkout([$class: 'GitSCM', 
                              branches: [[name: "*/${gitBranch}"]],  
                              userRemoteConfigs: [[url: gitRepo, 
                                                   credentialsId: "${env.GIT_CREDENTIALS_ID}"]]
                    ])
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
          registryCredential = 'dockerhub-credentials'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
    stage('Deploying React.js container to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deployment.yaml", 
                                         "service.yaml")
        }
      }
    }
  }
}