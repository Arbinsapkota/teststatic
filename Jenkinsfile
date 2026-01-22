

pipeline {
  agent any

  // Ensure Jenkins can find python.exe even if started without your user PATH.
  environment {
    // Add your Python install to the FRONT of PATH
    PATH = "C:\\Users\\arbin\\AppData\\Local\\Programs\\Python\\Python314\\;C:\\Users\\arbin\\AppData\\Local\\Programs\\Python\\Python314\\Scripts\\;${env.PATH}"
    PORT = "3000"
  }

  options { timestamps() }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Validate Files') {
      steps {
        bat '''
          if not exist index.html (echo index.html NOT FOUND! & exit /b 1)
          if not exist style.css  (echo style.css  NOT FOUND! & exit /b 1)
          echo Files validated successfully.
        '''
      }
    }

    stage('Kill Port (If Running) - CMD Only') {
      steps {
        bat '''
          echo Checking for processes on port %PORT%...
          for /f "tokens=5" %%p in ('netstat -ano ^| findstr ":%PORT%" ^| findstr /i "LISTENING"') do (
            echo Killing PID %%p on port %PORT%...
            taskkill /F /PID %%p >nul 2>&1
          )
          rem Always succeed, even if nothing was listening
          exit /b 0
        '''
      }
    }

    stage('Serve Website on Port') {
      steps {
        bat '''
          echo Detecting Python...
          set "PYCMD="
          where py >nul 2>nul && set "PYCMD=py -3"
          if not defined PYCMD where python >nul 2>nul && set "PYCMD=python"
          if not defined PYCMD (
            echo ERROR: Python not found in PATH!
            echo Current PATH:
            echo %PATH%
            exit /b 1
          )
          echo Python command: %PYCMD%

          for %%A in ("%WORKSPACE%") do set "WDIR=%%~fA"
          echo Starting server from %WDIR% on port %PORT% ...
          start "" /B cmd /c "cd /d %WDIR% && %PYCMD% -m http.server %PORT% > %TEMP%\\site-%PORT%.log 2>&1"

          rem Give the server a moment to bind the port
          timeout /t 2 /nobreak >nul

          echo ==================================================
          echo  WEBSITE LIVE at: http://localhost:%PORT%/
          echo  LOG FILE: %TEMP%\\site-%PORT%.log
          echo ==================================================
        '''
      }
    }
  }

  post {
    success {
      echo 'SUCCESS! Open http://localhost:' + env.PORT + '/'
    }
    failure {
      echo 'Build failed. Check Console Output and %TEMP%\\site-' + env.PORT + '.log'
    }
  }
}


// Example Jenkinsfile for Linux/MacOS systems using shell commands

// pipeline {
//   agent any

//   stages {

//     stage('Checkout Code') {
//       steps {
//         git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
//       }
//     }

//     stage('Validate Files') {
//       steps {
//         sh '''
//           test -f index.html || (echo "index.html NOT FOUND!" && exit 1)
//           test -f style.css  || (echo "style.css NOT FOUND!" && exit 1)
//         '''
//       }
//     }

//     stage('Serve Website on Port 3000') {
//       steps {
//         sh '''
//           pkill -f "python3 -m http.server 3000" || true
//           nohup python3 -m http.server 3000 >/tmp/website.log 2>&1 &
//           echo "Your site is LIVE at http://localhost:3000/"
//         '''
//       }
//     }
//   }

//   post {
//     success {
//       echo "DONE! Open http://localhost:3000/"
//     }
//   }
// }
