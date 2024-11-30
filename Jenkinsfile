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
        //stage('Checkout') {
        //    steps {
        //        git credentialsId: 'github-pat', url: 'https://github.com/BenUK78/example-spring-petclinic.git'
        //    }
        //}
        


        stage('Checkout') {
            environment {
                // Define variables for repository details
                GIT_REPO_URL = 'https://github.com/BenUK78/example-spring-petclinic.git'
                GIT_BRANCH = 'main'
            }
            steps {
                script {
                    try {
                        // Explicit checkout with verbose logging
                        checkout([
                            $class: 'GitSCM', 
                            branches: [[name: "*/${env.GIT_BRANCH}"]], 
                            userRemoteConfigs: [[
                                credentialsId: 'github-pat', 
                                url: "${env.GIT_REPO_URL}"
                            ]],
                            extensions: [
                                [$class: 'CleanCheckout'],
                                [$class: 'CloneOption', 
                                depth: 1, 
                                noTags: false, 
                                shallow: true
                                ]
                            ]
                        ])
                        
                        // Verification steps
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'git branch -v'
                        sh 'git log -1'
                    } catch (Exception e) {
                        // Comprehensive error logging
                        echo "Checkout FAILED: ${e.getMessage()}"
                        echo "Error Details:"
                        echo "Repository URL: ${env.GIT_REPO_URL}"
                        echo "Branch: ${env.GIT_BRANCH}"
                        echo "Credential ID: github-pat"
                        
                        // Additional diagnostic commands
                        sh 'env | grep -i git'
                        sh 'git config --list'
                        
                        // Rethrow to fail pipeline
                        throw e
                    }
                }
            }
        }
    



































        stage('Build') {
            steps {
                container('java') {
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