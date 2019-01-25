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
    # We need to use the debug version of the image as only
    # this variant contains a shell
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug-v0.7.0
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /root
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: regcred
          items:
            - key: .dockerconfigjson
              path: .docker/config.json
"""
}
  }
  stages {
    stage('Build with Kaniko') {
      environment {
        PATH = "/busybox:/kaniko:$PATH"
      }
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
            sh """#!/busybox/sh
            /kaniko/executor -f Dockerfile -c . --destination=docker-nexus.build-infra.forcarelabs.com/docker/jorian-kaniko:${env.GIT_COMMIT}
            """
        }
      }
    }
    stage('Deploy') {
      steps {
        container('kubectl') {
          sh("kubectl apply -f nginx.yaml")
        }
      }     
    }
  }
}
