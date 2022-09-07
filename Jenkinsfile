pipeline {
  agent {
    kubernetes {
      inheritFrom 'kubeagent'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: tools
    image: argoproj/argo-cd-ci-builder:latest
    command:
    - cat
    tty: true
"""
    }
  }
  stages {

    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
        GIT_CREDS = credentials('github')
      }
      steps {
        container('docker') {
          // Build new image
          sh "until docker ps; do sleep 3; done && docker build -t prachimittal2016/argocd-demo:${env.BUILD_NUMBER} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push prachimittal2016/argocd-demo:${env.BUILD_NUMBER}"
        }
      }
    }

    stage('Deploy to staging') {
      environment {
        GIT_CREDS = credentials('github')
      }
      steps {
        container('tools') {
          sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/PrachiMittal2016/argocd-demo-deploy.git"
          sh "git config --global user.email 'pracmittal@gmail.com'"
          sh "git config --global user.name 'PrachiMittal2016'"

          dir("argocd-demo-deploy") {
            sh "cd ./apps/demo-app/overlays/staging1/ && kustomize edit set image prachimittal2016/argocd-demo:${env.BUILD_NUMBER}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }

    stage('Deploy to QA') {
      steps {
        container('tools') {
          dir("argocd-demo-deploy") {
            sh "cd ./apps/demo-app/overlays/qa/ && kustomize edit set image prachimittal2016/argocd-demo:${env.BUILD_NUMBER}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }
    stage('Deploy to Prod') {
      steps {
        input message:'Approve deployment?'
        container('tools') {
          dir("argocd-demo-deploy") {
            sh "cd ./apps/demo-app/overlays/prod/ && kustomize edit set image prachimittal2016/argocd-demo:${env.BUILD_NUMBER}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }
  }
}
