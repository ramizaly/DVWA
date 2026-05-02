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
            credentialsId: 'github-dvwaa'
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

}