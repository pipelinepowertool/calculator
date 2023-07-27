pipeline {
  agent {
    label 'agent3'
  }
  stages {
    stage('Calculator Website Image build and Release') {
      steps {
        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS} docker.io"
        sh "docker build -t sdenboer/pipelinepowertool-calculator ."
        sh "docker push sdenboer/pipelinepowertool-calculator"
      }
    }
  }
}
