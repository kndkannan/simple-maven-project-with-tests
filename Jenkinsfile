pipeline {
  agent any

  parameters {
    booleanParam(name: 'fail_on_quality_gate_failure', defaultValue: true, description: 'Fail the build if the quality gate status is ERROR')
  }

  tools {
    // Install the Maven version configured as "M3" and add it to the path.
    maven "maven-3.9.6"
  }

  stages {
    stage('Build') {
      steps {
        // Get some code from a GitHub repository
        git 'https://github.com/kndkannan/simple-maven-project-with-tests.git'

        withMaven(maven: 'maven-3.9.6') {
          // Run Maven on a Unix agent.
          sh "mvn -Dmaven.test.failure.ignore=false clean package"
        }

      }

      post {
        // If Maven was able to run the tests, even if some of the test
        // failed, record the test results and archive the jar file.
        success {
          junit '**/target/surefire-reports/TEST-*.xml'
          archiveArtifacts 'target/*.jar'
        }
      }
    }
    stage('SonarQube Analysis') {
      steps {
        // Run SonarQube analysis using the SonarQube Scanner plugin
        timeout(time: 10, unit: 'MINUTES') {
          retry(2) {
            withSonarQubeEnv('local-sonar') {
              sh 'mvn sonar:sonar'
            }
          }
        }
        timeout(time: 5, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (params.fail_on_quality_gate_failure && qg.status == 'ERROR') {
              error "Quality Gate failed: ${qg.status}"
            } else {
              echo "Quality Gate status: ${qg.status}"
            }
          }
        }
      }
    }
  }

  post {
    always {
      script {
        def jobUrl = env.BUILD_URL
        def jobId = env.BUILD_ID
        def jobName = env.JOB_NAME
        slackSend(color: '#cf142b', channel: 'bootsy-devs', message: "Failed: Job NAME: ${JOB_NAME} #${jobId}\n ${jobUrl}" , token: '', failOnError : false )
      }
    }
  }
}
