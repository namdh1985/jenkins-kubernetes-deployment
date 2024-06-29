pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          name: podman-build
        spec:
          containers:
            - name: podman
              image: quay.io/podman/stable
              command:
                - tail
                - '-f'
                - '/dev/null'
              securityContext:
                privileged: true
              volumeMounts:
                - name: cgroup
                  mountPath: /sys/fs/cgroup
                - name: containers
                  mountPath: /var/lib/containers
                - name: podman-socket
                  mountPath: /run/podman/podman.sock
          volumes:
            - name: cgroup
              hostPath:
                path: /sys/fs/cgroup
            - name: containers
              hostPath:
                path: /var/lib/containers
            - name: podman-socket
              hostPath:
                path: /run/podman/podman.sock 
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
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          container('podman') {
            sh '''
            podman login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD docker.io
            podman build -t namdh1985/react-app:latest .
            podman push namdh1985/react-app:latest
            '''
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          kubectl apply -f deployment.yaml -n jenkins
          kubectl apply -f service.yaml -n jenkins
        '''
      }
    }
  }
}