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
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'deployment.yaml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'service.yaml',
                    enableConfigSubstitution: true
                )
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
                withKubeConfig([credentialsId: 'k8s-credentials', 
                               contextName: 'mmaaroju-k8s-cluster']) {
                    script {
                        sh 'echo  Deploy to Kubernetes!'
                        sh 'echo  kubectl set image deployment.apps/abstergo-deployment abstergo-container=${DOCKER_HUB_REPO}:${env.BUILD_ID}'
                        sh 'kubectl set image deployment.apps/abstergo-deployment abstergo-container=${DOCKER_HUB_REPO}:${env.BUILD_ID}'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
