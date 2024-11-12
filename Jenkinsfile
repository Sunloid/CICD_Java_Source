pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWScred')
        AWS_SECRET_ACCESS_KEY = credentials('AWScred')
    }
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
                // A seperate repository which contains the java source code.This was done for the simplicity of the repository 
                git branch: 'main', url:'https://github.com/Sunloid/CICD_Java_Source'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }


        stage("Build image and deploy it to ecr"){
            steps{
                script{
                     sh """
                        # Step 1: Authenticate Docker with Amazon ECR in the us-east-1 region
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 533267397460.dkr.ecr.ap-south-1.amazonaws.com
                        
                        # Step 2: Build the Docker image for the 'app' service with the specified build number
                        docker build -t app:${BUILD_NUMBER} ./app_dockerfile/.
                        
                        # Step 3: Build the Docker image for the 'web' service with the specified build number
                        docker build -t web:${BUILD_NUMBER} ./web_dockerfile/.
                        
                        # Step 4: Build the Docker image for the 'db' service with the specified build number
                        docker build -t db:${BUILD_NUMBER}  ./db_dockerfile/.
                        
                        # Step 5: Tag the 'app' Docker image with the ECR repository and build number
                        docker tag app:${BUILD_NUMBER} 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:app${BUILD_NUMBER}
                        
                        # Step 6: Tag the 'web' Docker image with the ECR repository and build number
                        docker tag web:${BUILD_NUMBER} 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:web${BUILD_NUMBER}
                        
                        # Step 7: Tag the 'db' Docker image with the ECR repository and build number
                        docker tag db:${BUILD_NUMBER} 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:db${BUILD_NUMBER}
                        
                        # Push the app image
                        docker push 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:app${BUILD_NUMBER}
                        
                        # Push the web image
                        docker push 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:web${BUILD_NUMBER}
                        
                        # Push the db image
                        docker push 533267397460.dkr.ecr.ap-south-1.amazonaws.com/cicd-repo:db${BUILD_NUMBER}

                    """
                }
            }
        }

        stage("Deploy to EC2"){
            steps {
		        script {
		            // ECS Cluster and Service configuration
		            def ecsCluster = "cicd-cluster1"
		            def appServiceName = "cicd-service2"  
		
		            // ECS Task Definitions
		            // def appTaskDefinition = "cicd-task"  
		
		            sh "aws ecs update-service --cluster ${ecsCluster} --region ap-south-1 --service ${appServiceName}  --force-new-deployment"
		
		    }
		 }
        }


    }
}