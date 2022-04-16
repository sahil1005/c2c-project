pipeline {
    agent any
    environment {
        CLUSTER_NAME = 'nodjs-test-cluster'
        AWS_ACCOUNT_ID="YOUR_ACCOUNT_ID"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="nodejs-test"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/sahil1005/c2c-project.git']]])     
            }
        }
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build ("${IMAGE_REPO_NAME}:${env.BUILD_ID}", "./app")
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${env.BUILD_ID} ${REPOSITORY_URI}:${env.BUILD_ID}"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${env.BUILD_ID}"
         }
        }
    }

    stage("kubernetes deployment"){
        steps{
          echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"  
                echo "Start deployment of deployment.yaml"
            step([$class: 'KubernetesEngineBuilder', clusterName: env.CLUSTER_NAME, location: env.AWS_DEFAULT_REGION, manifestPattern: 'deployment.yaml',  verifyDeployments: true])    
        }
    }

    stage("kubernetes service"){
        sh 'kubectl apply -f service.yml'
    }

    stage("kubernetes autoscalar"){
        sh 'kubectl apply -f autoscalar.yml'
    }         

}



