pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    tty: true
  - name: git
    image: alpine/git:latest
    command: [ "cat" ]
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
                container('git') {
                    git branch: 'master', url: 'https://github.com/Mubashir-web/web-template.git'
                }
            }
        }

        stage('Build & Push Image with Kaniko') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh """
                        mkdir -p /kaniko/.docker
                        echo "{\\"auths\\":{\\"https://index.docker.io/v1/\\":{\\"username\\":\\"$USER\\",\\"password\\":\\"$PASS\\"}}}" > /kaniko/.docker/config.json
                        /kaniko/executor --dockerfile=Dockerfile --context=. --destination=${REGISTRY}/${IMAGE}:${BUILD_NUMBER} --destination=${REGISTRY}/${IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                container('git') {
                    withCredentials([usernamePassword(credentialsId: 'gitops-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh '''
                        rm -rf gitops-basic
                        git clone https://$USER:$PASS@github.com/Mubashir-web/gitops-basic.git
                        cd gitops-basic/k8s
                        sed -i "s|image: .*|image: docker.io/mubashirck/my-gitops:'${BUILD_NUMBER}'|g" nginx-deployment.yaml
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
}

