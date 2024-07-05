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
              command: ["/bin/sh", "-c", "tail -f /dev/null"]
              securityContext:
                privileged: true
              volumeMounts:
                - name: cgroup
                  mountPath: /sys/fs/cgroup
                - name: containers
                  mountPath: /var/lib/containers
                - name: podman-socket
                  mountPath: /run/podman/podman.sock
            - name: kubectl
              image: bitnami/kubectl:latest
              securityContext:
                runAsUser: 0
              command:
                - tail
                - "-f"
                - /dev/null
              volumeMounts:
                - name: kubeconfig
                  mountPath: /.kube
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
            - name: kubeconfig
              emptyDir: {}
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
    stage('Build') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'harbor', usernameVariable: 'HARBOR_USERNAME', passwordVariable: 'HARBOR_PASSWORD')]) {
          container('podman') {
            sh '''
            systemctl restart podman
            podman login -u $HARBOR_USERNAME -p $HARBOR_PASSWORD core-harbor.f88.co
            podman build -t core-harbor.f88.co/library/react-app:latest .
            podman push core-harbor.f88.co/library/react-app:latest
            '''
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        container('kubectl') {          
          withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
              kubectl delete -f deployment.yaml -n jenkins
              kubectl delete -f service.yaml -n jenkins
            '''
          }
        }
      }
    }
  }
}