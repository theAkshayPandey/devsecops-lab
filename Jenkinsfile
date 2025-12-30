pipeline {
  agent any

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    /* =======================
       🔐 GATE 1 – SECRETS
       ======================= */
    stage('Secrets Scan - Gitleaks') {
      steps {
        sh '''
          echo "🔐 Running Gitleaks (Secrets Gate)"
          docker run --rm \
            -v $(pwd):/repo \
            zricethezav/gitleaks detect \
            --source=/repo \
            --exit-code 1
        '''
      }
    }

    /* =======================
       🧠 GATE 2 – SAST
       ======================= */
    stage('SAST - Semgrep') {
      steps {
        sh '''
          echo "🧠 Running Semgrep (High Confidence Only)"
          docker run --rm \
            -v $(pwd):/src \
            -w /src \
            returntocorp/semgrep \
            semgrep scan \
              --config=p/owasp-top-ten \
              --severity=ERROR
        '''
      }
    }

    /* =======================
       🏗 BUILD
       ======================= */
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t juice-shop-ci:latest .'
      }
    }

    /* =======================
       🐳 GATE 3 – IMAGE SCAN
       ======================= */
    stage('Image Scan - Trivy') {
      steps {
        sh '''
          echo "🐳 Running Trivy (CRITICAL vulns only)"
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy image \
              --severity HIGH, CRITICAL \
              --exit-code 1 \
              juice-shop-ci:latest
        '''
      }
    }

  }

  post {
    success {
      echo "✅ Build passed all security gates"
    }
    failure {
      echo "❌ Build failed due to security gate violation"
    }
  }
}
