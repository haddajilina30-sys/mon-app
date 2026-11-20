
pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    IMAGE = 'haddajilina/monapp'
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        bat 'docker version'
        bat "docker build -t %IMAGE%:%TAG% ."
      }
    }

    stage('Smoke Test') {
      steps {
        bat """
          docker rm -f monapp_test 2>nul || ver > nul
          docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
          ping -n 3 127.0.0.1 > nul
          curl -I http://localhost:8081 | find "200 OK"
          docker rm -f monapp_test
        """
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
          )
        ]) {
          powershell '''
            $env:PASS | docker login -u $env:USER --password-stdin
            if ($LASTEXITCODE -ne 0) { exit 1 }

            docker tag "$env:IMAGE`:$env:TAG" "$env:IMAGE`:latest"
            if ($LASTEXITCODE -ne 0) { exit 1 }

            docker push "$env:IMAGE`:$env:TAG"
            if ($LASTEXITCODE -ne 0) { exit 1 }

            docker push "$env:IMAGE`:latest"
            if ($LASTEXITCODE -ne 0) { exit 1 }
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Build+Test+Push OK'
    }

    failure {
      echo 'Build/Tests/Push KO'
    }
  }
}
