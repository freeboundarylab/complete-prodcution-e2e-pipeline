pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven'
    }
 
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/freeboundarylab/complete-prodcution-e2e-pipeline.git'
            }

        }

    }

}
