environment {
REGISTRY                 = credentials('REGISTRY_NAME')
DOCKER_REGISTRY_PASSWORD = credentials('DOCKER_REGISTRY_PASSWORD')
DOCKER_REGISTRY_USER     = credentials('DOCKER_REGISTRY_USERNAME')
GITHUB_CREDS             = credentials('GITHUB_CREDS')
}

stage('Get GIT_COMMIT') {
  steps {
  	script {
  		GIT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
  	}
  }
}

stage('docker-build') {
   options {
  	 timeout(time: 10, unit: 'MINUTES')
   }
   steps {
  	 sh '''#!/usr/bin/env bash
  	 echo "Shell Process ID: $$"
  	 docker login --user $DOCKER_REGISTRY_USER --	password $DOCKER_REGISTRY_PASSWORD
  	 docker build --tag ${REGISTRY}/sampleapp:${BRANCH}-${GIT_COMMIT} .
  	 docker push ${REGISTRY}/sampleapp:${BRANCH}-${GIT_COMMIT}
  	 '''
   }
}

  stage('Clone Helm Chart repo') {
      steps {
          dir('argo-cd') {
              git branch: 'main', credentialsId: 'GITHUB_CREDS', url: YOUR_SSH_REPO
              sh '''#!/usr/bin/env bash
                  git config --global user.email "jenkins-ci@gitlab.com"
                  git config --global user.name "jenkins-ci"
              '''
          }
      }
  }

stage('Deploy DEV') {
   when {
  	 branch 'dev'
   }
   script {
  	 steps {
  		 sh '''#!/usr/bin/env bash
  		 echo "Shell Process ID: $$"
  		 # Replace Repository and tag
  		 cd ./gitops-webapp-demo/deployment/dev/
  		 sed -r "s/^(\s*repository\s*:\s*).*/\1${REGISTRY}\/gitops-webapp-demo/" -i deployment.yaml
  		 sed -r "s/^(\s*tag\s*:\s*).*/\1${BRANCH}-${GIT_COMMIT}/" -i deployment.yaml
  		 git commit -am 'Publish new version' && git push || echo 'no changes'
  		 '''
  	 }
   }
}

stage('Deploy PROD') {
   when {
  	 branch 'prod'
   }
   script {
  	 steps {
  		 sh '''#!/usr/bin/env bash
  		 echo "Shell Process ID: $$"
  		 # Replace Repository and tag
  		 cd ./gitops-webapp-demo/deployment/dev/
  		 sed -r "s/^(\s*repository\s*:\s*).*/\1${REGISTRY}\/gitops-webapp-demo/" -i deployment.yaml
  		 sed -r "s/^(\s*tag\s*:\s*).*/\1${BRANCH}-${GIT_COMMIT}/" -i deployment.yaml
  		 git commit -am 'Publish new version' && git push || echo 'no changes'
  		 '''
  	 }
   }
}
