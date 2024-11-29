pipeline {
    agent {
        kubernetes {
            label 'java-builder'
            yaml '''
apiVersion: v1
kind: Pod
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
    
    environment {
        // Define environment variables
        //CONTAINER_IMAGE = 'petclinic'
        //CONTAINER_TAG = "${GIT_BRANCH.toLowerCase().replace('origin/', '')}-${GIT_COMMIT.substring(0,7)}"
        //CONTAINER_REGISTRY = 'docker.io/benuk78' // Replace with your registry
    }
    
    stages {

        stage('Bens Shell Stage') {
            steps {
                sh 'echo "Debug Pod Information"'
                sh 'pwd'
                sh 'ls -la'
                sh 'env'
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
        cleanup {
            // Clean up workspace
            cleanWs()
        }
    }
}