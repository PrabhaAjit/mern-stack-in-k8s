pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'prabhaajit'
        KUBE_CRED = 'kubeconfig-credentials-id'
        IMAGE_TAG = "${BUILD_ID}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/PrabhaAjit/mern-stack-in-k8s.git'
            }
        }

        stage('Build Images') {
            steps {
                withEnv(["TAG=${env.IMAGE_TAG}"]) {
                    sh 'docker-compose -f docker-compose.yaml build'
                }
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhubcredentials',
                                                  usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh 'echo "$P" | docker login -u "$U" --password-stdin'
                    sh """
                        docker push ${DOCKER_REGISTRY}/mern-server:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/mern-client:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: env.KUBE_CRED]) {
                    sh """
                        TAG=${IMAGE_TAG} envsubst < k8s/client.yaml | kubectl apply -f -
                        TAG=${IMAGE_TAG} envsubst < k8s/server.yaml | kubectl apply -f -
                        kubectl apply -f k8s/mongodb.yaml
                        kubectl apply -f k8s/ingress.yaml
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down --rmi local'
            sh '''
                docker system prune -f --filter "until=24h"
                docker volume prune -f
            '''
            cleanWs()
        }
    }
}
