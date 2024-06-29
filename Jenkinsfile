pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          name: docker-build
        spec:
          containers:
            - name: docker
              image: docker:19.03.12
              command:
                - tail
                - '-f'
                - '/dev/null'
              securityContext:
                privileged: true
              volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
          volumes:
            - name: docker-socket
              hostPath:
                path: /var/run/docker.sock
 
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
        container('docker') {
          sh 'docker build -t namdh1985/react-app:latest .'
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