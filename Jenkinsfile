pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/ramizaly/DVWA.git'
    ABORT_ON_QUALITY_GATE_FAILURE = 'true'  // Set to 'true' to block deployment on failure
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
        script {
          // Capture quality gate result without aborting
          def qg = waitForQualityGate()

          if (qg.status != 'OK') {
            // Post failure status to GitHub
            githubNotify status: 'FAILURE',
                         description: 'Quality Gate failed — fix issues before merging',
                         context: 'jenkins/quality-gate',
                         credentialsId: 'github-dvwa'

            echo "⚠️ Quality Gate FAILED — Status: ${qg.status}"

            // Abort or warn based on the variable
            if (env.ABORT_ON_QUALITY_GATE_FAILURE == 'true') {
              error "Pipeline aborted — Quality Gate failed."
            } else {
              echo "⚠️ Proceeding despite Quality Gate failure (ABORT_ON_QUALITY_GATE_FAILURE=false)"
              unstable "Quality Gate failed but continuing pipeline."
            }

          } else {
            // Post success status to GitHub
            githubNotify status: 'SUCCESS',
                         description: 'Quality Gate passed',
                         context: 'jenkins/quality-gate',
                         credentialsId: 'github-dvwa'
            echo "✅ Quality Gate PASSED"
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        echo "🚀 Deploying application from branch ${env.BRANCH_NAME}..."
        echo "🚀 Deployment complete. (Simulated)"
      }
    }
  }

/*
  post {
    success {
      echo "✅ Pipeline completed successfully on ${env.BRANCH_NAME}."
    }
    unstable {
      echo "⚠️ Pipeline completed with warnings — Quality Gate failed but deployment ran."
    }
    failure {
      echo "❌ Pipeline failed on ${env.BRANCH_NAME}."
    }
  }
*/
}