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
            steps {
                git 'https://github.com/spring-projects/spring-petclinic.git'
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