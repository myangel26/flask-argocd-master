pipeline {

  agent {
    kubernetes{
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          serviceAccountName: jenkins-admin
          containers:
            - name: python
              image: python:3.8-slim-buster
              command:
              - cat
              tty: true
              volumeMounts:
              - mountPath: /root/.cache
                name: python-cache
          resources:
            requests:
              memory: "300Mi"
              cpu: "500m"
              ephemeral-storage: "2.5Gi"
            limits:
              memory: "600Mi"
              cpu: "1" 
          volumes:
            - name: python-cache
              hostPath:
                path: /tmp
      '''
    }
  }
  
  // None: khia báo agent khi ta chạy từng stage
  // khai báo ở đây thì chạy chung nguyên stage

  environment {
    DOCKER_IMAGE = "truongphamxuan/flask-docker"
    CREDENTIAL_ID = "docker-account"
    KUBERNETES_CONFIG = "kube-config"
    NAMESPACE = "flask-argocd"
    DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
    GITHUB_EMAIL = "truongphamxuan2604@gmail.com"
    GITHUB_NAME = "truong"
    GITHUB_ACC = "${credentials('github-account').username}"
    GITHUB_PWD = "${credentials('github-account').password}"
  }

  options {
    timestamps()
    timeout(time: 15, unit: 'MINUTES') // sau 180 phút, pipeline sẽ được hủy
    ansiColor('xterm')
    disableConcurrentBuilds()
    // buildDiscarder(logRotator(numToKeepStr: '250', daysToKeepStr: '5'))
  }

  stages{

    stage("ECHO"){
      steps {
        script{
          sh "echo ${DOCKER_TAG}"
        }
      }
    }

    stage("TEST"){
      steps {
        container('python') {
          sh "pip install poetry" 
          sh "poetry install"
          sh "poetry run pytest"
        }
      }
    }

    stage('Get GIT_COMMIT') {
      steps {
        script {
          GIT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
          sh "echo GIT_COMMIT: ${GIT_COMMIT}"
        }
      }
    }

    // stage('docker-build') {
    //   options {
    //     timeout(time: 120, unit: 'SECONDS')
    //   }

    //   steps {
    //     sh '''#!/usr/bin/env bash
    //       echo "Shell Process ID: $$"
    //       git config --global user.email ${GITHUB_EMAIL}
    //       git config --global user.name ${GITHUB_NAME}
    //       echo pwd
    //       rm -rf flask-argocd-k8s
    //       git clone https://$GITHUB_ACC:$GITHUB_PWD@github.com/myangel26/flask-argocd-k8s.git
    //       cd flask-argocd-k8s
    //       ls -la
    //     '''
    //     container('docker'){
    //       sh "docker build -t ${DOCKER_IMAGE}:${GIT_COMMIT} . "
    //       withCredentials([usernamePassword(credentialsId: CREDENTIAL_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
    //         sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
    //         sh "docker push ${DOCKER_IMAGE}:${GIT_COMMIT}"
    //       }
    //       // clean to save disk
    //       sh "docker image rm ${DOCKER_IMAGE}:${GIT_COMMIT}"
    //     }
    //   }
    // }

    stage('Deploy DEV') {
      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
        '''
      }
    }

    // stage("BUILD") {
    //   steps {
    //     container('docker'){
    //       sh "echo $DOCKER_TAG"
    //       sh "docker build --network=host -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
    //       sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
    //       sh "docker image ls | grep ${DOCKER_IMAGE}"
    //       withCredentials([usernamePassword(credentialsId: CREDENTIAL_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]){
    //         // sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
    //         sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
    //         sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
    //         sh "docker push ${DOCKER_IMAGE}:latest"
    //       }
    //       // clean to save disk
    //       sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
    //       sh "docker image rm ${DOCKER_IMAGE}:latest"
    //     }
    //   }
    // }

    stage('Cleanup Workspace'){
        steps {
            script {
                cleanWs()
            }
        }
    }

  }

  post{
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }

}