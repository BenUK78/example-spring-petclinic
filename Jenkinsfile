		pipeline {
		    agent any
		    
		    stages {
		        stage ("checkout") {
		            steps {
		                echo "Checkout stage"
		                sh "ls"
		                git branch: 'main', url: 'https://github.com/BenUK78/example-spring-petclinic'
		                sh "ls"
		            }
		        }
		        
		        stage ("build") {
		            steps {
		                echo "Build stage"
		                //sh "./mvnw package"
		            }
		        }
		        
		        stage("Parallel Tests") {                                                                  
		            parallel {
		                stage('testsA') {
		                    steps {
		                        sh "echo test set A"
		                        sleep 5
		                    }
		                }
		                stage('testsB') {
		                    steps {
		                        sh "echo test set B"
		                        sleep 4
		                    }
		                }
		                stage('testsC') {
		                    steps {
		                        sh "echo test set C"
		                        sleep 5
		                    }
		                }
		            }
		        }
		        
		        stage ("capture") {
		            steps {
		                echo "Capture stage"
		                //archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
		                //jacoco()
		                //junit stdioRetention: '', testResults: '**/target/surefire-reports/TEST*.xml'
		            }
		        }
		        
		        stage ("notify") {
		            steps {
		                // Send Email
		                echo "Send Email stage"
		                emailext(
		                    body: "${env.BUILD_URL}\n${currentBuild.absoluteUrl}",
		                    to: 'always@foo.bar',
		                    recipientProviders: [[$class: 'RequesterRecipientProvider']],
		                    subject: "${currentBuild.currentResult}: Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
		                )
		            }
		        }
		    }
}
