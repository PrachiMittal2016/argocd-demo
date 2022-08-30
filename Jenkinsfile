podTemplate(containers: [
    containerTemplate(
        name: 'dind', 
        image: 'docker:18.09-dind', 
        securityContext:
          privileged: 'true'
     ),
    containerTemplate(
        name: 'docker', 
        image: 'docker:18.09', 
        command: 'cat', 
        tty: 'true'),
    containerTemplate(
        name: 'tools', 
        image: 'argoproj/argo-cd-ci-builder:v0.13.1', 
        command: 'cat', 
        tty: 'true')
  ]) {

      node(POD_LABEL) {
        stage('Build') {
            environment {
                DOCKERHUB_CREDS = credentials('dockerhub')
                GIT_CREDS = credentials('github')
            }
            git 'https://github.com/PrachiMittal2016/argocd-demo.git', branch: 'lenovo-Jenkins'
            container('docker') {
                stage('Build project') {
                    sh "until docker ps; do sleep 3; done && docker build -t PrachiMittal2016/argocd-demo:1 ."
                    // Publish new image
                    sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push PrachiMittal2016/argocd-demo:1"
                    //sh '''
                    //echo "m here"
                    //until docker ps; do sleep 3; done && docker build -t PrachiMittal2016/argocd-demo:1 .
                    // Publish new image
                    //docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push PrachiMittal2016/argocd-demo:1
                    //'''
                }
            }
        }

        stage('Deploy to Staging') {
            environment {
                GIT_CREDS = credentials('github')
            }
            git url: 'https://github.com/PrachiMittal2016/argocd-demo-deploy.git', branch: 'test'
            container('tools') {
                stage('Deploy') {
                    sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/PrachiMittal2016/argocd-demo-deploy.git"
                    sh "git config --global user.email 'ci@ci.com'"
                    //sh '''
                    //git clone -b test https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/PrachiMittal2016/argocd-demo-deploy.git
                    //git config --global user.email 'ci@ci.com'
                    //echo "in deploy stage"
                    //'''
                    dir("argocd-demo-deploy") {
                        sh "cd ./staging && kustomize edit set image PrachiMittal2016/argocd-demo:1"
                        sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
                    }
                }
            }
        }

    }
}
