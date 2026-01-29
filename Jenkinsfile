pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
    timeout(time: 20, unit: 'MINUTES')
  }

  environment {
    CONTAINER_NAME = 'site2'
    PORT           = '4000'
  }

  // Poll every minute (remove once you enable a GitHub webhook)
  triggers { pollSCM('* * * * *') }

  stages {
    stage('Checkout & Info') {
      steps {
        // Jenkins already checks out (Pipeline from SCM),
        // these commands just give useful context in logs.
        bat 'git --version & docker --version'
        bat 'git log -1 --oneline'
      }
    }

    stage('Get Commit Hash') {
      steps {
        script {
          env.COMMIT_HASH = bat(
            label: 'Get short commit hash',
            script: '@echo off\r\nfor /f %%a in (\'git rev-parse --short HEAD\') do @echo %%a',
            returnStdout: true
          ).trim()
          echo "Commit: ${env.COMMIT_HASH}"
        }
      }
    }

    stage('Build Image') {
      steps {
        // Use cache for faster builds (no --no-cache)
        bat 'docker build -t %CONTAINER_NAME%:%COMMIT_HASH% .'
        bat 'docker tag %CONTAINER_NAME%:%COMMIT_HASH% %CONTAINER_NAME%:latest'
      }
    }

    stage('Deploy') {
      steps {
        bat 'docker rm -f %CONTAINER_NAME% 2>NUL || echo no existing container'
        bat 'docker run -d -p %PORT%:80 --name %CONTAINER_NAME% %CONTAINER_NAME%:%COMMIT_HASH%'
      }
    }

    stage('Verify') {
      steps {
        // Robust, single-line curl on Windows.
        // IMPORTANT: %%{http_code} (double %) escapes properly in CMD.
        bat 'curl --fail --silent --show-error --location -o NUL -w "HTTP %%{http_code}\\n" http://localhost:%PORT%'
      }
    }
  }

  post {
    success {
      echo "✅ Deployed: http://localhost:${PORT}"
    }
    failure {
      echo "❌ Deployment failed. Container logs:"
      bat 'docker logs --tail 200 %CONTAINER_NAME% || echo no container logs'
    }
  }
}