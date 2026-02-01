pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/mubashirck"
        IMAGE = "my-gitops"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mubashir-web/gitops-basic.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${REGISTRY}/${IMAGE}:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh '''
                git clone https://github.com/your/gitops-repo.git
                cd gitops-repo/k8s
                sed -i "s|image: .*|image: ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}|g" nginx-deployment.yaml
                git config --global user.email "jenkins@ci.local"
                git config --global user.name "Jenkins CI"
                git commit -am "Update image to ${BUILD_NUMBER}"
                git push origin main
                '''
            }
        }
    }
}

