pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/ramizaly/DVWA.git'
    // No BRANCH needed — Multibranch Jenkins sets env.BRANCH_NAME automatically
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: "${env.BRANCH_NAME}",
            url: "${REPO_URL}",
            credentialsId: 'github-dvwa'
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv('sonar') {
          withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh '''
              sonar-scanner \
                -Dsonar.projectKey=DVWA-demo \
                -Dsonar.sources=. \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_TOKEN
            '''
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }

  post {
    failure {
      echo "Quality Gate failed on branch ${env.BRANCH_NAME} — merge to main is blocked."
    }
    success {
      echo "Quality Gate passed on ${env.BRANCH_NAME}. Safe to merge."
    }
  }
}