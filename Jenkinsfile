pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:20.10.7-dind
    command:
    - cat
    tty: true
"""
        }
    }

    environment {
        REGISTRY = "docker.io/mubashirck"
        IMAGE = "my-gitops"
    }

    stages {
        stage('Checkout Web Repo') {
            steps {
                git branch: 'master', url: 'https://github.com/Mubashir-web/web-template.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}
                    docker tag ${REGISTRY}/${IMAGE}:${BUILD_NUMBER} ${REGISTRY}/${IMAGE}:latest
                    docker push ${REGISTRY}/${IMAGE}:latest
                    """
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitops-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    rm -rf gitops-basic
                    git clone https://$USER:$PASS@github.com/Mubashir-web/gitops-basic.git
                    cd gitops-basic/k8s
                    sed -i "s|image: .*|image: ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}|g" nginx-deployment.yaml
                    git config --global user.email "jenkins@ci.local"
                    git config --global user.name "Jenkins CI"
                    git commit -am "Update image to ${BUILD_NUMBER}" || echo "No changes to commit"
                    git push origin master
                    '''
                }
            }
        }
    }
}

