pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            app: test
          namespace: jenkins  
        spec:
          containers:
          - name: git
            image: bitnami/git:latest
            command:
            - cat
            tty: true
          - name: kubectl
            image: joshendriks/alpine-k8s
            command:
             - /bin/cat
            tty: true 
          - name: maven
            image: maven:3.8.3-adoptopenjdk-11
            command:
            - cat
            tty: true
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug-51734fc3a33e04f113487853d118608ba6ff2b81
            command:
            - cat
            tty: true
            volumeMounts:
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          volumes:
          - name: kaniko-secret
            secret:
              secretName: dockercred
              items:
                - key: .dockerconfigjson
                  path: config.json
      '''
    }      
  }
  environment{
    DOCKERHUB_USERNAME = "duniaalk"
    APP_NAME = "kaniko-webapp-demo"
    IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }
 stages {
    stage('Checkout SCM') {
      steps {
        container('git') {
          git url: 'https://github.com/kunchalavikram1427/maven-employee-web-application.git',
          branch: 'master'
        }
      }
    }
    stage('Build SW'){
      steps {
        container('maven'){
          sh 'mvn -Dmaven.test.failure.ignore=true clean package'
        }
      }
    }
    stage('Build Container Image'){
      steps {
        container('kaniko'){
          sh "/kaniko/executor --context $WORKSPACE --destination duniaalk/kaniko-maven:$IMAGE_TAG"
        }
      }
    }
    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" kaniko-maven.yaml'
            sh 'kubectl apply -f kaniko-maven.yaml'
          }
        }
      }
    }
  }
}
