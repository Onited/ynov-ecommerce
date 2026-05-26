pipeline {
  agent any
  tools { nodejs 'node-20' }

  environment {
    NODE_ENV = 'test'
  }

  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Ensure libatomic') {
      steps {
        sh '''
          set -euo pipefail
          if ldconfig -p 2>/dev/null | grep -q libatomic; then
            echo "libatomic already present"
            exit 0
          fi

          echo "libatomic missing — attempting to download and extract locally"
          mkdir -p libs

          # try apt-get download (no root required for download)
          if command -v apt-get >/dev/null 2>&1; then
            apt-get download libatomic1 || apt-get download libatomic || true
          fi

          DEB=$(ls libatomic1*.deb 2>/dev/null || true)
          if [ -z "$DEB" ]; then
            ARCH=$(uname -m)
            case "$ARCH" in
              x86_64) PKG_ARCH=amd64 ;;
              aarch64|arm64) PKG_ARCH=arm64 ;;
              *) PKG_ARCH=amd64 ;;
            esac
            # best-effort fallback URL (may need adjustment for specific distro)
            URL="https://deb.debian.org/debian/pool/main/g/gcc-12/libatomic1_12.2.0-14_${PKG_ARCH}.deb"
            echo "Trying $URL"
            curl -fsSL "$URL" -o libatomic.deb || true
            DEB=$(ls libatomic*.deb 2>/dev/null || true)
          fi

          if [ -n "$DEB" ]; then
            dpkg-deb -x "$DEB" libs
            echo "extracted $DEB into libs/"
          else
            echo "Could not download libatomic deb — later stages may still fail"
          fi
        '''
      }
    }
    stage('Install') {
      steps {
        sh '''
          # add any extracted libs to LD_LIBRARY_PATH for this job
          if [ -d "$PWD/libs" ]; then
            LD_DIRS=$(find "$PWD/libs" -type d -maxdepth 3 -print | paste -sd ":" -)
            export LD_LIBRARY_PATH="$LD_DIRS:$LD_LIBRARY_PATH"
            echo "LD_LIBRARY_PATH set to $LD_LIBRARY_PATH"
          fi
          node -v || true
          npm -v || true
          npm ci
        '''
      }
    }
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