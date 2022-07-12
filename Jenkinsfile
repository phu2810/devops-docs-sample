pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  parameters{
     string(name:'TAG_NAME',defaultValue: '',description:'')
  }
  environment {
    DOCKERHUB_CREDENTIAL_ID = 'dockerhub-id'
    GITHUB_CREDENTIAL_ID = 'github-id'
    KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
    DOCKERHUB_NAMESPACE = 'phutest123'
    GTIHUB_ACCOUNT = 'phu2810'
    APP_NAME = 'devops-docs-sample'
  }
  stages {
    stage('checkout scm') {
      steps {
        checkout(scm)
      }
    }
    stage('build & push snapshot') {
      steps {
        container('maven') {
          sh 'docker build -t docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKERHUB_CREDENTIAL_ID" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER '
          }
        }
      }
    }
    stage('push latest'){
       when{
         branch 'master'
       }
       steps{
         container('maven'){
           sh 'docker tag  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
           sh 'docker push  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
         }
       }
    }
    stage('deploy to dev') {
      when{
        branch 'master'
      }
      steps {
        container('maven') {
          withCredentials([
            kubeconfigFile(
              credentialsId: env.KUBECONFIG_CREDENTIAL_ID,
              variable: 'KUBECONFIG')
            ]) {
              sh 'envsubst < deploy/dev/docs-sample.yaml | kubectl apply -f -'
              sh 'envsubst < deploy/dev/docs-sample.yaml | kubectl apply -f -'
          }
        }
      }
    }
  }
}
