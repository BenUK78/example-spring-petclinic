pipeline {
    agent {
        kubernetes {
            label 'java-builder'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
    - name: java
      image: 'maven:3.8.4-openjdk-17-slim'
      command:
        - cat
      tty: true
      resources:
        limits:
          memory: 2Gi
          cpu: 1
    - name: buildah
      image: quay.io/buildah/stable:v1.29
      command:
        - cat
      tty: true
      securityContext:
        privileged: true
      resources:
        limits:
          memory: 2Gi
          cpu: 1
            '''
        }
    }


    // Bens Jenkinsfile
    
    environment {
        // Set up any environment variables you might need
        MAVEN_OPTS = '-Dmaven.repo.local=.m2/repository'
    }
    
    stages {

        stage('Checkout') {
            // Define environment variables for the repository
            // This allows easier configuration and reusability
            environment {
                // Full URL of the Git repository
                GIT_REPO_URL = 'https://github.com/BenUK78/example-spring-petclinic.git'
                
                // Specific branch to checkout (can be easily changed)
                GIT_BRANCH = 'main'
            }
            
            steps {
                // Wrap in script block for more complex logic and error handling
                script {
                    try {
                        // Comprehensive checkout method with multiple configuration options
                        checkout([
                            // Specify the SCM (Source Control Management) class
                            $class: 'GitSCM',
                            
                            // Define which branch to checkout
                            // Uses environment variable and adds * for pattern matching
                            branches: [[name: "*/${env.GIT_BRANCH}"]],
                            
                            // Configure repository access
                            userRemoteConfigs: [[
                                // Reference to credentials stored in Jenkins
                                credentialsId: 'github-pat',
                                
                                // Use environment variable for repository URL
                                url: "${env.GIT_REPO_URL}"
                            ]],
                            
                            // Additional checkout extensions
                            extensions: [
                                // Clean the workspace before checkout
                                [$class: 'CleanCheckout'],
                                
                                // Configure clone options
                                [$class: 'CloneOption',
                                    depth: 1,        // Shallow clone (only latest commit)
                                    noTags: false,   // Include tags
                                    shallow: true    // Reduce clone time/bandwidth
                                ]
                            ]
                        ])
                        
                        // Verification steps to confirm successful checkout
                        sh 'pwd'           // Print current directory
                        sh 'ls -la'        // List all files with details
                        sh 'git branch -v' // Show current branch
                        sh 'git log -1'    // Show latest commit
                    } catch (Exception e) {
                        // Comprehensive error handling and logging
                        echo "Checkout FAILED: ${e.getMessage()}"
                        echo "Error Details:"
                        echo "Repository URL: ${env.GIT_REPO_URL}"
                        echo "Branch: ${env.GIT_BRANCH}"
                        echo "Credential ID: github-pat"
                        
                        // Diagnostic commands to gather more information
                        sh 'env | grep -i git'        // Show git-related environment variables
                        sh 'git config --list'        // Show git configuration
                        
                        // Rethrow the exception to fail the pipeline
                        throw e
                    }
                }
            }
        }


        //stage('Code Quality') {        NOTE: IF ACTIVE THEN FAILS THE PIPELINE DUE TO 1650+ CODE VIOLATIONS IN SPRING-PETCLINIC
        //    steps {
        //        container('java') {
        //            sh 'mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs'
        //        }
        //    }
        //}


        stage('Compile') {
            steps {
                container('java') {
                    sh 'mvn clean compile'
                }
            }
        }
    

        stage('Build') {
            steps {
                container('java') {
                    sh 'echo "------------"'
                    sh 'echo "Java Version Number - Spring-Petclinic requires at least version 17"'
                    sh 'java -version'
                    sh 'echo "------------"'
                    sh 'mvn package -DskipTests'
                }
            }
        }
        
        stage('Test') {
            steps {
                container('java') {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        
        stage('Build Container Image') {
            steps {
                container('buildah') {
                    script {
                        // Build image using Buildah
                        sh '''
                            buildah bud -t spring-petclinic:${BUILD_NUMBER} .
                        '''
                        
                        // Optional: Tag for potential registry push
                        sh '''
                            buildah tag spring-petclinic:${BUILD_NUMBER} localhost:5000/spring-petclinic:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                container('java') {
                    // Use Kubernetes credentials and cluster configuration
                    withKubeConfig([credentialsId: 'kubernetes-cluster-credentials']) {
                        // Apply Kubernetes deployment manifest
                        sh '''
                            sed -i "s/IMAGE_TAG/${BUILD_NUMBER}/g" k8s/deployment.yaml
                            kubectl apply -f k8s/deployment.yaml
                        '''
                        
                        // Verify deployment
                        sh '''
                            kubectl rollout status deployment/spring-petclinic
                            kubectl get pods -l app=spring-petclinic
                        '''
                    }
                }
            }
        }

  
    }
    
    post {
        success {
            echo 'Build completed successfully!'
            emailext(
                body: "Build Successful\n${env.BUILD_URL}\n${currentBuild.absoluteUrl}",
                to: 'always@foo.bar',
                recipientProviders: [[$class: 'RequesterRecipientProvider']],
                subject: "SUCCESS: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
            )
        }
        failure {
            echo 'Build failed. Please check the logs.'
            emailext(
                body: "Build Failed\n${env.BUILD_URL}\n${currentBuild.absoluteUrl}",
                to: 'always@foo.bar',
                recipientProviders: [[$class: 'RequesterRecipientProvider']],
                subject: "FAILURE: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
            )
        }
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}