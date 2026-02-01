pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/mubashirck"
        IMAGE = "my-gitops"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Mubashir-web/gitops-basic.git'
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

    stage('Update GitOps Repo') { steps { withCredentials([usernamePassword(credentialsId: 'gitops-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) { sh ''' git clone https://$USER:$PASS@github.com/Mubashir-web/gitops-basic.git cd gitops-basic/k8s sed -i "s|image: .*|image: ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}|g" nginx-deployment.yaml git config --global user.email "jenkins@ci.local" git config --global user.name "Jenkins CI" git commit -am "Update image to ${BUILD_NUMBER}" git push origin main || git push origin master ''' } } }
}
}

