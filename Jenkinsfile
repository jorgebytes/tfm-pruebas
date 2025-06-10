pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "ddatdt12/python-web-app"
  }

  stages {
    stage("test") {
      agent {
          docker {
            image 'python:3.9-slim-buster'
            args '-u 0:0 -v /python/tmp:/root/.cache'
          }
      }
      steps {
        sh "apt-get update && apt-get install make"
        sh "make test"
      }
    }

    stage("build") {
      agent { node {label 'master'}}
      environment {
        // This is the tag of the image that will be pushed to the registry
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
        }

        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
        script{
          if (env.BRANCH_NAME == 'master') {
            sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            sh "docker push ${DOCKER_IMAGE}:latest"
            
            sh "docker image rm ${DOCKER_IMAGE}:latest"
          }
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
      }
    }

    stage("deploy") {
      agent { node {label 'master'}}
      when { branch "main" }
      steps {
        // sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa"
        sh "docker pull ${DOCKER_IMAGE}:latest"
        sh "docker run --rm -it -p 5000:5000 ${DOCKER_IMAGE}:latest"
      }
    }
  }

  post {
    success {
      echo "SUCCESSFULL"
    }
    failure {
      echo "FAILED"
    }
  }
}