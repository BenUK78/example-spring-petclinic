pipeline {

    agent {

        kubernetes {

            // Use the Java agent template defined in your JCasC configuration

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

'''

        }

    }

    

    environment {

        // Define environment variables

        DOCKER_IMAGE = 'petclinic'

        DOCKER_TAG = "${GIT_BRANCH.toLowerCase().replace('origin/', '')}-${GIT_COMMIT.substring(0,7)}"

        DOCKER_REGISTRY = 'docker.io/yourusername' // Replace with your Docker Hub username or private registry

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

        

        stage('Build Docker Image') {

            steps {

                script {

                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")

                }

            }

        }

        

        stage('Push Docker Image') {

            steps {

                script {

                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {

                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()

                    }

                }

            }

        }

        

        stage('Deploy to Kubernetes') {

            steps {

                script {

                    // Use kubectl to deploy to Kubernetes

                    sh """

                        kubectl apply -f k8s/deployment.yaml

                        kubectl set image deployment/petclinic petclinic=${DOCKER_IMAGE}:${DOCKER_TAG}

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