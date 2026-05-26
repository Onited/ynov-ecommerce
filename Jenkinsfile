pipeline {
  agent { 
    docker { image 'node:20-alpine' } 
  }

  environment {
    NODE_ENV = 'test'
  }

  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Install') { 
      steps { 
        sh 'npm ci' 
      } 
    }
    
    stage('Lint') { 
      steps { 
        sh 'npm run lint' 
      } 
    }
    
    stage('Test') {
      steps {
        sh 'npm test -- --coverage'
      }
      post {
        always {
          junit 'reports/junit.xml'
          archiveArtifacts artifacts: 'coverage/**'
        }
      }
    }
    
    stage('Deploy') {
      when { 
        branch 'main' 
      }
      steps { 
        sh './deploy.sh' 
      }
    }
  }

  post {
    success { 
      echo '✅ Pipeline OK' 
    }
    failure { 
      mail to: 'team@x.io', 
           subject: "❌ Echec: ${env.JOB_NAME}", 
           body: "Le build ${env.BUILD_NUMBER} a échoué.\n\nVoir les logs : ${env.BUILD_URL}"
    }
  }
}