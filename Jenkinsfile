pipeline {
    agent{
        label "slave"
    }
    stages{
        stage("Git Checkout"){
            steps{
                git branch: 'main', url: 'https://github.com/gerarahul/sample--project.git'
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
                    nexusUrl: '13.233.140.223:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'demoapp-release', 
                    version: '1.0.8'
                }
            }
        }
        stage('Docker image build'){
            steps{
                script{
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:latest'
                }
            }
        }   
        stage("Push image to docker hub"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'DOCKER-CRED', url: 'https://hub.docker.com/repository/docker/rgera0901/demo_application_repo')  {
                        sh 'docker image push rgera0901/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image push rgera0901/$JOB_NAME:latest'
                    }
                }
            }
        } 
    }
}


