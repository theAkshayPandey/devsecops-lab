pipeline {
  agent any

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Secrets Scan - Gitleaks') {
      steps {
        sh 'docker run --rm -v $(pwd):/repo zricethezav/gitleaks detect --source=/repo --exit-code 1'
      }
    }

    stage('SAST - Semgrep') {
      steps {
        sh 'docker run --rm -v $(pwd):/src returntocorp/semgrep semgrep scan --config=p/owasp-top-ten --error'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t juice-shop-ci:latest .'
      }
    }

    stage('Image Scan - Trivy') {
      steps {
        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity HIGH,CRITICAL --exit-code 1 juice-shop-ci:latest'
      }
    }

  }
}
