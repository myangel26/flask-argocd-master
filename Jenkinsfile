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
            - name: docker
              image: docker:latest
              command:
              - cat
              tty: true
              volumeMounts:
              - mountPath: /var/run/docker.sock
                name: docker-sock
          resources:
            requests:
              memory: "300Mi"
              cpu: "500m"
            limits:
              memory: "600Mi"
              cpu: "1" 
          volumes:
            - name: python-cache
              hostPath:
                path: /tmp
            - name: docker-sock
              hostPath:
                path: /var/run/docker.sock
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
    CREDENTIAL_GITHUB = "github-account-token"
  }

  options {
    timestamps()
    timeout(time: 15, unit: 'MINUTES') // sau 180 phút, pipeline sẽ được hủy
    ansiColor('xterm')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '3', daysToKeepStr: '3', artifactNumToKeepStr: '3', artifactDaysToKeepStr: '3'))
  }

  stages{

    stage("ECHO"){
      steps {
        script{
          sh "echo ${DOCKER_TAG}"
        }
      }
    }

    // stage("TEST"){
    //   steps {
    //     container('python') {
    //       sh "pip install poetry" 
    //       sh "poetry install"
    //       sh "poetry run pytest"
    //     }
    //   }
    // }

    stage('Get GIT_COMMIT') {
      steps {
        script {
          DK_TAG = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
          sh "echo DK_TAG: ${DK_TAG}"
        }
      }
    }

    stage('docker-build') {
      options {
        timeout(time: 120, unit: 'SECONDS')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
        '''
        container('docker'){
          sh "docker build -t ${DOCKER_IMAGE}:${DK_TAG} . "
          withCredentials([usernamePassword(credentialsId: CREDENTIAL_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DK_TAG}"
          }
          // clean to save disk
          sh "docker image rm ${DOCKER_IMAGE}:${DK_TAG}"
        }
      }
    }

    stage("INSTALL KUBECTL"){
      steps{
        withKubeConfig([credentialsId: "${KUBERNETES_CONFIG}"]) {
          // sh 'curl -LO "https://dl.k8s.io/release/`curl -LS https://dl.k8s.io/release/stable.txt`/bin/linux/amd64/kubectl"'
          sh 'curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash'
          // sh 'chmod u+x ./kubectl'
          // sh './kubectl version'
          sh 'pwd'
          sh 'ls -la'
        }
      }
    }

    stage('Deploy DEV') {
      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          git config --global user.email ${GITHUB_EMAIL}
          git config --global user.name ${GITHUB_NAME}
          pwd
        '''
       withCredentials([usernamePassword(credentialsId: CREDENTIAL_GITHUB, passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
          script {
            sh """
              rm -rf flask-argocd-k8s
              git clone https://\$GITHUB_USERNAME:\$GITHUB_PASSWORD@github.com/myangel26/flask-argocd-k8s.git
              git branch --show-current
              cd ./flask-argocd-k8s/overlays/dev && ../../../kustomize edit set image \${DOCKER_IMAGE}=\${DOCKER_IMAGE}:\${DK_TAG}
              echo ">>>>>>>>> ${DK_TAG}"
            """
            // ls -la
            // git commit -am 'Publish new version' && git push origin master || echo 'no changes'
          }
        }
      }
    }
    // ./kubectl kustomize ../flask-argocd-k8s/overlays/dev/ edit set image ${DOCKER_IMAGE}=${DOCKER_IMAGE}:${GIT_COMMIT}
    // cd ./flask-argocd-k8s/overlays/dev && ../../../kubectl kustomize edit set image ${DOCKER_IMAGE}=${DOCKER_IMAGE}:${GIT_COMMIT}

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