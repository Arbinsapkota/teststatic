
pipeline {
  agent any
  options { timestamps() }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
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

    stage('Kill Port 3000 (If Running)') {
      steps {
        // Force a zero exit code even if nothing is listening / access denied, etc.
        bat '''
          echo Checking if port 3000 is in use...
          powershell -NoProfile -Command " ^
            $c = Get-NetTCPConnection -LocalPort 3000 -State Listen -ErrorAction SilentlyContinue; ^
            if ($c) { ^
              try { Stop-Process -Id $c.OwningProcess -Force -ErrorAction SilentlyContinue } catch {} ^
            }; ^
            exit 0"
          echo Port 3000 cleaned (or was not in use).
        '''
      }
    }

    stage('Serve Website on Port 3000') {
      steps {
        bat '''
          echo Detecting Python...
          set "PYCMD="
          where py >nul 2>nul && set "PYCMD=py -3"
          if not defined PYCMD where python >nul 2>nul && set "PYCMD=python"
          if not defined PYCMD (
            echo ERROR: Python not found in PATH!
            exit /b 1
          )
          echo Python detected: %PYCMD%

          rem Start background server from workspace
          for %%A in ("%WORKSPACE%") do set "WDIR=%%~fA"
          echo Starting server from %WDIR% ...
          start "" /B cmd /c "cd /d %WDIR% && %PYCMD% -m http.server 3000 > %TEMP%\\site-3000.log 2>&1"

          rem give it a second to bind the port
          timeout /t 2 /nobreak >nul

          echo ==================================================
          echo  WEBSITE SHOULD BE LIVE AT: http://localhost:3000/
          echo  LOG FILE: %TEMP%\\site-3000.log
          echo ==================================================

          rem Show which process is listening (if any). Do not fail build if none found.
          powershell -NoProfile -Command "Get-NetTCPConnection -LocalPort 3000 -State Listen -ErrorAction SilentlyContinue | Select-Object -First 1 | ForEach-Object { 'Listening PID: ' + $_.OwningProcess } ; exit 0"
        '''
      }
    }
  }

  post {
    success {
      echo 'SUCCESS! Open http://localhost:3000/'
    }
    failure {
      echo 'Build failed. Check Console Output and %TEMP%\\site-3000.log'
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
