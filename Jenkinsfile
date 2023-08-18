pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "nxtruong451999/docker-flask"
    SSH_KEY="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCmdym5/BFcPkgMXKvfxJQai91su9at7L5YEH+ukwBP4vchEeyhMcsmv0WkKBybc1R+V9Dt/8IUMmUx6q2aWznm0nXVP9qEU4hfHwil48tMDwB8g17c3hHa+8+VsCHzisjRhM9CkJkkqF+MYnVai9VGBtoPF8wrMPyCszK522qJsYFE4FmEoFiixr00D6urEKdo4QqmbxrETHvqnxcYrbfloOREAOa2PprfjnuZCSH8WsUMV73Md05MJl6GAFWFS+0U+gGm93wR1aSCCJSfFpTN51A3d7GIomVxBjD6gen9zlL7IfdZECGsNDashYmS3KlD18lTQ7tAK6MM5wZiQ9nL7T9rO94FZbIn4rZXyknNBBZujd4z5RXlOJmxWGCQv+nZAcFFw/zXDxvFzlpFGwo8MP1XBO+rjkar3ANxmVqJahSYNEKM6cnkONW9FhtHfvzwyQ93Z7m/T+zBz9ImmmZXANPB00E0YsRchiyA2APoMJa/EDIOLcmSxJTrytCaG8A1sOpniKqdOh9QfoelXTlX1F8TTKiQNUxa2Tm7Y/VpAHwolZioJW7mg4zlwl9654O5BcHxWf/EAO4yPnMqJN9YKmzocU9XMahX9GF8QzjTQMITwwPGueQn1tYfxgukhtNYl9hJLabTMTOKDtKklzTS6HV+xjbW0e965UM30/sAiw== nguyenxuantruongee@gmail.com"
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
    stage("SSH server") {
      agent { node {label 'built-in'}}
      steps {
        sshagent(['ssh-key']) {
          sh 'ssh -o StrictHostKeyChecking=no -l root 137.184.15.239 touch test.txt'
        }
      }
    }
    stage("Deploy") {
        agent { node { label 'built-in' } }
        steps {
            script {
            sshagent(['ssh-key']) {
                sh "scp -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa deploy.sh root@137.184.15.239:/root/demo_flask"
                sh "ssh -o StrictHostKeyChecking=no -i $SSH_KEY root@137.184.15.239 'chmod +x /root/demo_flask/deploy.sh && /root/demo_flask/deploy.sh'"
            }
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
