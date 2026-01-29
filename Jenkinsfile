pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "mariemsouadi12189"
        FRONTEND_IMAGE = "project-frontend"
        BACKEND_IMAGE  = "project-backend"
    }

    stages {

        stage("Checkout Code") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mariemsouadi123/Ecommerce-Platform-with-DevOps-Cloud-Practices.git'
            }
        }

        stage("Build Frontend Image") {
            steps {
                script {
                    docker.build(
                        "${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}",
                        "frontend"
                    )
                }
            }
        }

        stage("Build Backend Image") {
            steps {
                script {
                    docker.build(
                        "${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER}",
                        "backend"
                    )
                }
            }
        }

        stage("Push Images to Docker Hub") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                        docker tag ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:latest
                        docker tag ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${BACKEND_IMAGE}:latest

                        docker push ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:latest

                        docker push ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USER}/${BACKEND_IMAGE}:latest
                    """
                }
            }
        }

        stage("Deploy to Kubernetes (kubectl)") {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh """
                        export KUBECONFIG=\$KUBECONFIG_FILE

                        kubectl apply -f k8s/mysql.yaml
                        kubectl apply -f k8s/backend.yaml
                        kubectl apply -f k8s/frontend.yaml

                        kubectl rollout status deployment/backend  -n ecommerce
                        kubectl rollout status deployment/frontend -n ecommerce
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ pipeline successful!"
        }
        failure {
            echo "❌ pipeline failed!"
        }
    }
}

