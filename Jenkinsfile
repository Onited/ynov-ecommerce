pipeline {
  agent any

  environment {
    NODE_ENV = 'test'
  }

  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Install') { steps { sh 'npm ci' } }
    stage('Lint')    { steps { sh 'npm run lint' } }
    stage('Test') {
      steps {
        sh 'npm run test:coverage'
      }
      post {
        always {
          archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
        }
      }
    }
    stage('Deploy') {
      when {
        allOf {
          branch 'main'
          expression { fileExists('deploy.sh') }
        }
      }
      steps {
        sh 'chmod +x ./deploy.sh && ./deploy.sh'
      }
    }
  }

  post {
    success { echo '✅ Pipeline OK' }
    failure { echo "❌ ${env.JOB_NAME} a échoué" }
  }
}