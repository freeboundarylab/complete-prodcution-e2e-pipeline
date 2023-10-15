pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven'
    }

    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "lulgabc"
        DOCKER_PASS = 'docker'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("Jenkins_API_Token")

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

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage("Docker Build and Push") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Modify deployment image TAG") {
            steps {
                sh """
	                cat deployment.yaml
                	sed -i 's/lulgabc\/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                	cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
	                git config --global user.name "lulg"
                    git config --global user.email "freeboundarylab@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Build Tag for Deployment"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName:'Default')]) {
                    sh "git push https://github.com/freeboundarylab/complete-prodcution-e2e-pipeline main"
                }
            }
        }

    }

}
