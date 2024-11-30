pipeline {
    agent {
        kubernetes {
            label 'java-builder'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/dynamic-agent: "true"
spec:
  containers:
  - name: maven
    image: maven:3.8.1-openjdk-11-slim
    command:
    - cat
    tty: true
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
  - name: buildah
    image: quay.io/buildah/stable:v1.30.0
    command:
    - cat
    tty: true
    securityContext:
      privileged: true
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
'''
        }
    }
   
    stages {
        stage('Debug Information') {
            steps {
                container('maven') {
                    sh 'echo "Maven Container"'
                    sh 'mvn --version'
                    sh 'pwd'
                    sh 'ls -la'
                }
                container('buildah') {
                    sh 'echo "Buildah Container"'
                    sh 'buildah version'
                }
            }
        }
    }
   
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}