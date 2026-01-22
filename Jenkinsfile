

pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        // Uses the same repo that provided this Jenkinsfile
        checkout scm
      }
    }

    stage('Validate Files') {
      steps {
        bat '''
          if not exist index.html (echo index.html NOT FOUND! & exit /b 1)
          if not exist style.css  (echo style.css NOT FOUND!  & exit /b 1)
          echo Files present: index.html and style.css
        '''
      }
    }

    stage('Serve Website on Port 3000') {
      steps {
        // 1) Free port 3000 if already used
        bat '''
          powershell -NoProfile -Command ^
            "$c = Get-NetTCPConnection -LocalPort 3000 -State Listen -ErrorAction SilentlyContinue; ^
             if ($c) { try { Stop-Process -Id $c.OwningProcess -Force -ErrorAction SilentlyContinue } catch {} }"
        '''

        // 2) Start Python HTTP server in the background from the workspace
        // Prefer the Python launcher (py -3). If you don't have it, replace with "python".
        bat '''
          for %%A in ("%WORKSPACE%") do set WDIR=%%~fA
          echo Starting local server from %WDIR% ...
          start "" /B cmd /c "cd /d %WDIR% && py -3 -m http.server 3000 > %TEMP%\\site-3000.log 2>&1"
          echo ==================================================
          echo  SITE SHOULD BE LIVE AT: http://localhost:3000/
          echo  (log: %TEMP%\\site-3000.log)
          echo ==================================================
        '''
      }
    }
  }

  post {
    success {
      echo 'Done. Open http://localhost:3000/'
    }
    failure {
      echo 'Build failed. See Console Output for details.'
    }
  }
}






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
