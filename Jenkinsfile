pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'mmaaroju/abstergo-portal'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: ' https://github.com/mmaaroju/edurekaDevopsProject2.git'
            }
        }

        stage('Build Docker Image') {
             when {
                branch 'master'
            }

            steps {
               script {
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_ID}")
                }
             }
         }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
         }

        stage('Deploy to Kubernetes') {
            steps {
                 kubernetesDeploy(
                    kubeconfigId: 'k8s-credentials',
                    configs: 'deployment.yaml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'k8s-credentials',
                    configs: 'service.yaml',
                    enableConfigSubstitution: true
                )
                }
            }
        }


    post {
        always {
            cleanWs()
        }
    }
}
