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
        ACCESS_KEY = credentials('aws_access_key_id')
        SECRET_KEY = credentials('aws_secret_access_key')
    }
    stages{
        stage("Git Checkout"){
            steps{
                git branch: 'main', url: 'https://github.com/gerarahul/Java_springboot_app_deployment.git'
            }
        }
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
                    }
                }
            }
        }
    }
}
