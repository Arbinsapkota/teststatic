
pipeline {
  agent any

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/Arbinsapkota/docker.git'
      }
    }

    stage('Validate Files') {
      steps {
        sh '''
          test -f index.html || (echo "index.html NOT FOUND!" && exit 1)
          test -f style.css  || (echo "style.css NOT FOUND!" && exit 1)
        '''
      }
    }

    stage('Serve Website on Port 3000') {
      steps {
        sh '''
          pkill -f "python3 -m http.server 3000" || true
          nohup python3 -m http.server 3000 >/tmp/website.log 2>&1 &
          echo "Your site is LIVE at http://localhost:3000/"
        '''
      }
    }
  }

  post {
    success {
      echo "DONE! Open http://localhost:3000/"
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
