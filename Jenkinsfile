pipeline {
  agent {
    label 'agent3'
  }
  stages {
    stage('Calculator Website Image build and Release') {
      steps {
        pipelinePowerToolInitiator()
        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS} docker.io"
        sh "docker build --platform='linux/arm64' -t sdenboer/pipelinepowertool-calculator ."
        sh "docker push sdenboer/pipelinepowertool-calculator"
      }
    }
  }
  post {
    always {
      pipelinePowerToolElasticPublisher(userName: "elastic", password: "WYVI+2L0ZjI3n11PjTNP", hostName: "192.168.1.163", port: 9200)
    }
  }
}
