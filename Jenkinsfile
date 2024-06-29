pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          name: node-app
        spec:
          containers:
            - name: node
              image: node:19-alpine3.16
              command: ["tail", "-f", "/dev/null"]  
          restartPolicy: Never  
      '''
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
    stage('Build react app') {
      steps {
        container('node') {
          sh 'npm i'
        }
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