pipeline {
    agent any 

    stages {
        stage('Build .NET Webapp') {
            steps {
                script {
                    // Create .NET webapp
                    sh 'dotnet new webapp'
                    
                    // Build the webapp
                    sh 'dotnet build -c Release'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build docker image
                    sh 'docker build -t webapp .'
                }
            }
        }
        stage('Push Docker Image to ECR') {
          steps {
            script {
              // Load AWS parameters from configuration file
              def props = readProperties file: 'aws-config.properties'
            
                withCredentials([[
                  $class: 'AmazonWebServicesCredentialsBinding',
                  credentialsId: 'my-aws-credentials',
                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                  ]]) {
                  // Login to ECR
                  sh "$(aws ecr get-login --region ${props.AWS_REGION} --no-include-email)"
                
                  // Tag the Docker image
                  sh "docker tag webapp:latest ${props.AWS_ACCOUNT}.dkr.ecr.${props.AWS_REGION}.amazonaws.com/webapp:latest"
                
                  // Push the Docker image to ECR
                  sh "docker push ${props.AWS_ACCOUNT}.dkr.ecr.${props.AWS_REGION}.amazonaws.com/webapp:latest"
                  }
              }
          }
      }

        stage('Deploy to ECS') {
            steps {
                script {
                    // Load AWS parameters from configuration file
                    def props = readProperties file: 'aws-config.properties'

                    // Deploy Docker container to ECS
                    sh "aws ecs update-service --cluster ${props.ECS_CLUSTER} --service ${props.ECS_SERVICE} --force-new-deployment"
                }
            }
        }
    }
}
