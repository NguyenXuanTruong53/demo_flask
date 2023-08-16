pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "nhtua/flask-docker"
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
      agent { label 'origin' }
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        script {
          docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").build(".")
          docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
          docker.image("${DOCKER_IMAGE}:latest").tag("${DOCKER_TAG}")
          docker.image("${DOCKER_IMAGE}:latest").push()
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
