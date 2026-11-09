pipeline {
  // Сборка идёт внутри сборочного контейнера (ci/Dockerfile.build).
  // Сокет docker монтируем, чтобы из контейнера собрать и запушить образ приложения.
  agent {
    dockerfile {
      dir 'ci'
      filename 'Dockerfile.build'
      args '-u root -v /var/run/docker.sock:/var/run/docker.sock '
    }
  }

  parameters {
    string(name: 'DEPLOY_DOCKER_HOST', defaultValue: '',
           description: 'Куда деплоить: пусто = docker хоста Jenkins, либо ssh://user@host')
  }

  environment {
    REGISTRY  = 'localhost:5001'              
    IMAGE     = "${REGISTRY}/demo-app"
    TAG       = "${env.BUILD_NUMBER}"
    CONTAINER = 'demo-app'
    HOST_PORT = '8081'                        
  }

  options { timestamps() }

  triggers { pollSCM('H/1 * * * *') }

  stages {
    // Двухстадийный Dockerfile сам компилирует исходники (maven) и кладёт jar в рантайм-образ
    stage('Build docker image') {
      steps {
        sh '''
          docker build -t $IMAGE:$TAG -t $IMAGE:latest .
          docker push $IMAGE:$TAG
          docker push $IMAGE:latest
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          if [ -n "$DEPLOY_DOCKER_HOST" ]; then export DOCKER_HOST="$DEPLOY_DOCKER_HOST"; fi
          docker pull $IMAGE:$TAG
          docker rm -f $CONTAINER || true
          docker run -d --name $CONTAINER --restart unless-stopped -p ${HOST_PORT}:8080 $IMAGE:$TAG
        '''
      }
    }
  }

  post {
    always { sh 'docker image prune -f || true' }
  }
}
