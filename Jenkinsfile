pipeline {
  agent {
    docker {
      image 'python:3.5.1'
    }

  }
  stages {
    stage('Code pull') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        withEnv(overrides: ["HOME=${env.WORKSPACE}"]) {
          sh '''
              python -m pip install --user virtualenv
              python -m virtualenv test_app
              source test_app/bin/activate
              python -m pip install -r requirements/dev.txt
             '''
        }
      }
    }
    stage('Static code metrics') {
      steps {
        echo "Raw metrics"
        sh  ''' 
                radon raw --json app > raw_report.json
                radon cc --json app > cc_report.json
                radon mi --json app > mi_report.json
                sloccount --duplicates --wide irisvmpy > sloccount.sc
            '''
        echo "Style check"
        sh  ''' 
                pylint app || true
            '''
      }
      post{
        always{
          step([$class: 'CoberturaPublisher',
          autoUpdateHealth: false,
          autoUpdateStability: false,
          coberturaReportFile: 'reports/coverage.xml',
          failNoReports: false,
          failUnhealthy: false,
          failUnstable: false,
          maxNumberOfBuilds: 10,
          onlyStable: false,
          sourceEncoding: 'ASCII',
          zoomCoverageChart: false])
        }
      }
    }
  }

  post {
    always {
      echo 'Completed'

    }
    failure {
      echo 'Send e-mail, when failed'
    }
  }
}
