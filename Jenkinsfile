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
  // None: khia báo agent khia chạy từng stage
  // khai báo ở đây thì chạy chung nguyên stage

  environment {
    DOCKER_IMAGE = "truongphamxuan/flask-docker"
    CREDENTIAL_ID = "docker-account"
    KUBERNETES_CONFIG = "kube-config"
    NAMESPACE = "flask-project"
    DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
  }

  stages{

    stage('Cleanup Workspace'){
        steps {
            script {
                cleanWs()
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