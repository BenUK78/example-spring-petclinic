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
        CONTAINER_IMAGE = 'petclinic'
        CONTAINER_TAG = "${GIT_BRANCH.toLowerCase().replace('origin/', '')}-${GIT_COMMIT.substring(0,7)}"
        CONTAINER_REGISTRY = 'docker.io/benuk78' // Replace with your registry
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        
        stage('Test') {
            steps {
                container('maven') {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Container Image') {
            steps {
                container('buildah') {
                    script {
                        sh """
                            buildah bud \
                            -t ${CONTAINER_REGISTRY}/${CONTAINER_IMAGE}:${CONTAINER_TAG} \
                            -f Dockerfile .
                        """
                    }
                }
            }
        }
        
        stage('Push Container Image') {
            steps {
                container('buildah') {
                    script {
                        withCredentials([usernamePassword(
                            credentialsId: 'container-registry-credentials', 
                            usernameVariable: 'REGISTRY_USER', 
                            passwordVariable: 'REGISTRY_PASS'
                        )]) {
                            sh """
                                buildah login -u ${REGISTRY_USER} -p ${REGISTRY_PASS} docker.io
                                buildah push ${CONTAINER_REGISTRY}/${CONTAINER_IMAGE}:${CONTAINER_TAG}
                            """
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update deployment with new image
                    sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl set image deployment/petclinic petclinic=${CONTAINER_REGISTRY}/${CONTAINER_IMAGE}:${CONTAINER_TAG}
                    """
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
        cleanup {
            // Clean up workspace
            cleanWs()
        }
    }
}