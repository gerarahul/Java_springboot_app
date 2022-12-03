pipeline {
    agent{
        label "slave"
    }
    parameters {
        choice(name: 'action', choices: 'create\ndestroy\ndestroyekscluster', description: 'create/update or destroy eks cluster')
        string(name: 'cluster', defaultValue: 'prod_ekscluster', description: 'Eks cluster name')
        string(name: 'region', defaultValue: 'ap-south-1', description: 'Eks cluster region')
    }
    environment {
        DOCKERHUB_CREDENTIALS= credentials('docker_credentials')
        ACCESS_KEY = credentials('aws_access_key_id')
        SECRET_KEY = credentials('aws_secret_access_key')
    }
    stages{
        stage("Git Checkout"){
            steps{
                git branch: 'main', url: 'https://github.com/gerarahul/Java_springboot_application.git'
            }
        }
        stage("Unit Testing"){
            steps{
                sh "mvn test"
            }
        }
        stage("Integration testing"){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'Sonar-api-key'){
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('Quality gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-api-key'
                }
            }
        }
        stage('Upload jar file to nexus'){
            steps{
                script{
                    nexusArtifactUploader artifacts: 
                    [
                        [
                        artifactId: 'springboot', 
                        classifier: '', file: 'target/Uber.jar', 
                        type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus-jenkins-authentiation', 
                    groupId: 'com.example', 
                    nexusUrl: '13.233.167.76:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'demoapp-release', 
                    version: '1.3.0'
                }
            }
        }
        stage('Docker image build'){
            steps{
                script{
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:latest'
                    echo 'Docker Image build succesfully'
                }
            }
        }
        stage('Login to Docker Hub'){      
            steps{
                sh 'docker logout'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                 
                echo 'Login Completed'                
            }           
        }  
        stage("Push image to docker hub"){
            steps{
                sh 'docker push rgera0901/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker push rgera0901/$JOB_NAME:latest'
                echo "Image pushed to docker_hub successfully"
                }
        }
        // CI process completed now CD process will be done on kubernets
        stage("eks connect"){
            steps{
                sh """
                    aws configure set aws_access_key_id "${ACCESS_KEY}"
                    aws configure set aws_secret_access_key "${SECRET_KEY}"
                    aws configure set region ""
                    aws eks --region ${params.region} update-kubeconfig --name ${params.cluster}
                    
                    """
                echo "connection to eks cluster is succesfuly happened"
            }
        }

        stage("Deployment On Eks"){
            when {expression { params.action == 'create'}}
            steps{
                script{
                    def apply = false
                    try{
                        input message: 'please confirm the apply to initiate the deployments', ok: 'ready to apply the config'
                        apply = true
                    }
                    catch(err){
                        apply = false
                        CurrentBuild.result = "UNSTABLE"
                    }
                    if(apply){
                        sh "kubectl apply -f ."
                        echo "Deployed on Eks cluster"
                    }
                }
            }
        }
        stage("Delete Deployments"){
            when {expression { params.action == 'destroy'}}
            steps{
                script{
                    def destroy = false
                    try{
                        input message: 'please confirm the destroy to delete the deployments', ok: 'ready to destroy the config'
                        destroy = true
                    }
                    catch(err){
                        destroy = false
                        CurrentBuild.result = "UNSTABLE"
                    }
                    if(destroy){
                        sh "kubectl delete -f ."
                        echo "Deployments is deleted or destroyed"
                    }
                }
            }
        }
    }
}


