pipeline {
  agent {
    kubernetes {
      label 'my-kubernetes-agent'
    }
  }
  stages {
    stage('Clone repository') {
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
    stage('Build and push Docker image') {
      steps {
        sh 'docker build -t my-image:latest .'
        sh 'docker push my-image:latest'
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        kubernetesDeploy(
          configs: 'kubernetes/deployment.yaml',
          kubeconfigId: 'my-kubeconfig'
        )
      }
    }
  }
}