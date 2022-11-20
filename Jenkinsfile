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

                    def readPomVersion = readMavenPom file: 'pom.xml'

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"

                    nexusArtifactUploader artifacts: 
                    [
                        [
                        artifactId: 'springboot', 
                        classifier: '', 
                        file: 'target/Uber.jar', 
                        type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus-jenkins-authentiation', 
                    groupId: 'com.example', 
                    nexusUrl: '13.233.140.223:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'demoapp-release', 
                    version: "${readPomVersion.version}"

                }
            }
        }
    }
}