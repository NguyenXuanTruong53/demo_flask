pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "nxtruong451999/docker-flask"
    SSH_KEY="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCeN2zonSS0J++s2xA+Ah0RyrKGLqQiYXUMBJJ+7oYhfTkD7+e2ruV57wwS4sKj494QxKgvC5nGW+tFNUbEcYHks+OKb3Zpt4mcoGEWOj7pG/mzq/1X6N3BA13U0a8rw9o+qCCak73VTT5zhBEcBj+XX591ypJJG4g1NHgYBVaOsIG+DNXp5LBR6nvHCLR9JJwNIwJD+j4gHnxbup5wm8C5IUSg8KcctkSOw/SdFdTzXlTOnUSFv3gMTpbms/pKQrNxDXT3+lXVszyiNvnDHDvv02TroCi01cvShA5lKkFW7CxoMAPHJLBsiheHVdzyIQkCZrYn8+rjQzpS37kLODSDzZCBQiTfrnfkF6D1UCUJnQYUXY9IZI5Bow5GM+dGSltoINYrkZ4CyrgXxYQ7B6HQY1MFfOxzRTGht8oDzqy51WONJPsqJOxLKMMSJzeVsPKIyXK703Q76ypcNNrgRGBpgglAtQvOBVQSXHbxZXRtk3nE3s+K/BHtr5NkND1yXos= root@ubuntu-s-1vcpu-1gb-sfo3-01"
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
    stage("Deploy") {
      agent { node {label 'built-in'}}
      steps {
        script {
            def sshKey = credentials('ssh-key')
            sshUserPrivateKey(credentialsId: 'ssh-key', sshKeyVariable: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCeN2zonSS0J++s2xA+Ah0RyrKGLqQiYXUMBJJ+7oYhfTkD7+e2ruV57wwS4sKj494QxKgvC5nGW+tFNUbEcYHks+OKb3Zpt4mcoGEWOj7pG/mzq/1X6N3BA13U0a8rw9o+qCCak73VTT5zhBEcBj+XX591ypJJG4g1NHgYBVaOsIG+DNXp5LBR6nvHCLR9JJwNIwJD+j4gHnxbup5wm8C5IUSg8KcctkSOw/SdFdTzXlTOnUSFv3gMTpbms/pKQrNxDXT3+lXVszyiNvnDHDvv02TroCi01cvShA5lKkFW7CxoMAPHJLBsiheHVdzyIQkCZrYn8+rjQzpS37kLODSDzZCBQiTfrnfkF6D1UCUJnQYUXY9IZI5Bow5GM+dGSltoINYrkZ4CyrgXxYQ7B6HQY1MFfOxzRTGht8oDzqy51WONJPsqJOxLKMMSJzeVsPKIyXK703Q76ypcNNrgRGBpgglAtQvOBVQSXHbxZXRtk3nE3s+K/BHtr5NkND1yXos= root@ubuntu-s-1vcpu-1gb-sfo3-01')
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
