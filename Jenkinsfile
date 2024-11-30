pipeline {
    agent {
        kubernetes {
            // Use the Java agent template we defined in the JCasC configuration
            label 'java-builder'
            // Optionally, you can override or add to the template here
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                some-label: some-label-value
            spec:
              containers:
              - name: java
                image: maven:3.8.1-openjdk-11-slim
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
    

        stage('Build') {
            steps {
                container('java') {
                    sh 'echo "------------"'
                    sh 'echo "Java Version Number - Spring-Petclinic requires at least 17"'
                    sh 'java -version'
                    sh 'echo "------------"'
                    sh 'mvn clean package -DskipTests'
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
        
        stage('Code Quality') {
            steps {
                container('java') {
                    sh 'mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed. Please check the logs.'
        }
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}