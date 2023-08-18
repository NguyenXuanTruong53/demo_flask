pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "nxtruong451999/docker-flask"
  }

  stages {
    stage("Test") {
      agent {
          docker {
            image 'python:3.8-slim-buster'
            args '-u 0:0 -v /tmp:/root/.cache'
          }
      }
      steps {
        sh "pip install poetry"
        sh "poetry install"
        sh "poetry run pytest"
        // sh 'apt-get install -y curl'
        // sh "apt-get update && apt-get install -y docker.io"
        // sh 'curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose'
        // sh 'chmod +x /usr/local/bin/docker-compose'
        // sh 'chmod +x ~/docker-compose'
        // sh 'sudo mv ~/docker-compose /usr/local/bin/docker-compose'
      }
    }

    stage("build") {
      agent { node {label 'built-in'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        withCredentials([usernamePassword(credentialsId: 'c4a30513-ef7a-4e86-a96d-b157dd28c128', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
            sh "docker push ${DOCKER_IMAGE}:latest"
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
        sh "docker image rm ${DOCKER_IMAGE}:latest"
      }
    }
    stage("Deploy") {
      steps {
        // withCredentials([sshKey(credentialsId: 'ssh-key', sshKeyVariable: 'SSH_KEY')]) {
        //     sh "ssh -i $SSH_KEY root@137.184.15.239 './deploy.sh'"
        // }
        script {
            def sshKey = credentials('ssh-key')  // Replace 'ssh-key' with your actual credentials ID
            sshUserPrivateKey(credentialsId: sshKey.id, keyFileVariable: 'SSH_KEY')
            sh "ssh -i $SSH_KEY root@137.184.15.239 './deploy.sh'"
        }
      }
    }
  }

  post {
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }
}
