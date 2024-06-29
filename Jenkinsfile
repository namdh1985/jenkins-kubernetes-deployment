pipeline {
  environment {
    dockerImage = "docker:19.03.12"
    GIT_CREDENTIALS_ID = 'github-credentials'
  }
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:          
          - name: docker
            image: ${dockerImage}
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
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
        container ('docker') {
          sh 'docker build -t namdh1985/react-app:latest .'
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