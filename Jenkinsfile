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
            /* steps {
                withKubeConfig([credentialsId: 'k8s-credentials', contextName: 'your-k8s-cluster']) {
                    script {
                        sh 'kubectl set image deployment/abstergo-deployment abstergo-container=${DOCKER_HUB_REPO}:${env.BUILD_ID}'
                    }
                }
            } */

           /* steps {
                withCredentials([file(credentialsId: 'k8s-credentials', variable: 'KUBECONFIG')]) {
                 sh 'kubectl apply -f deployment.yaml'
                 sh 'kubectl apply -f service.yaml'
                 // sh 'kubectl set image deployment.apps/abstergo-deployment abstergo-container=${DOCKER_HUB_REPO}:${env.BUILD_ID}'
                }
                
              } */
            
            steps {
              withCredentials([string(credentialsId: 'k8s-credentials', variable: 'KUBECONFIG_CONTENT')]) {
              sh '''
                echo "Debug: KUBECONFIG_CONTENT starts with:"
                echo "${KUBECONFIG_CONTENT:0:100}"  # Print first 100 characters for debugging
                printf "%s" "$KUBECONFIG_CONTENT" > kubeconfig_temp
                export KUBECONFIG=kubeconfig_temp
                kubectl apply -f deployment.yaml
                rm kubeconfig_temp
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
