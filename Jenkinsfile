pipeline {
  agent any
  stages {
    stage('Calculator Website Image build and Release') {
      steps {
        pipelinePowerToolInitiator()
        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS} docker.io"
        sh 'docker buildx ls | grep "raspberry-builder" || docker buildx create --name raspberry-builder --platform "linux/arm64,linux/arm/v7,linux/arm/v6" --driver "docker-container" --driver-opt network=host'
        sh 'docker buildx use raspberry-builder'
        sh "docker buildx build --platform='linux/arm64' -t sdenboer/pipelinepowertool-calculator . --push"
//         sh "docker push sdenboer/pipelinepowertool-calculator"
      }
    }
  }
  post {
    always {
      pipelinePowerToolElasticPublisher(userName: "elastic", password: "WYVI+2L0ZjI3n11PjTNP", hostName: "192.168.1.163", port: 9200)
    }
  }
}
