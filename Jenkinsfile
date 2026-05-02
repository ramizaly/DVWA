pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/ramizaly/DVWA.git'
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
      when {
        anyOf {
          branch 'Dev'
          triggeredBy 'UserIdCause'
        }
      }
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
      when {
        anyOf {
          branch 'Dev'
          triggeredBy 'UserIdCause'
        }
      }
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }

 post {
    success {
      publishChecks name: 'jenkins/quality-gate',
                   status: 'COMPLETED',
                   conclusion: 'SUCCESS',
                   title: 'Quality Gate',
                   summary: 'Quality Gate passed — safe to merge'
      echo "✅ Quality Gate passed on ${env.BRANCH_NAME}."
    }
    failure {
      publishChecks name: 'jenkins/quality-gate',
                   status: 'COMPLETED',
                   conclusion: 'FAILURE',
                   title: 'Quality Gate',
                   summary: 'Quality Gate failed — fix issues before merging'
      echo "❌ Qualityy Gate failed on ${env.BRANCH_NAME} — PR blocked."
    }
  }
}