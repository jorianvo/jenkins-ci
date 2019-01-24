pipeline {
  agent {
    kubernetes {
      label 'agent'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins
  containers:
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Deploy') {
      steps {
        container('kubectl') {
          sh("kubectl apply -f nginx.yaml")
        }
      }     
    }
  }
}
