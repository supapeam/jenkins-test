pipeline {
  agent any

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  environment {
    PYTHONPATH = "${WORKSPACE}/src"
    PIP_DISABLE_PIP_VERSION_CHECK = '1'
  }

  stages {
    stage('Setup venv') {
      steps {
        sh '''#!/usr/bin/env bash
        set -euo pipefail
        python3 -m venv .venv
        . .venv/bin/activate
        pip install -q --upgrade pip
        pip install -q -r requirement.txt
        python --version
        '''
      }
    }

    stage('Lint') {
      steps {
        sh '''#!/usr/bin/env bash
        set -euo pipefail
        . .venv/bin/activate
        ruff check src tests
        '''
      }
    }

    stage('Test') {
      steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
          sh '''#!/usr/bin/env bash
          set -euo pipefail
          . .venv/bin/activate
          pytest -v --junitxml=report.xml
          '''
        }
      }
      post {
        always {
          junit 'report.xml'
        }
      }
    }

    stage('Package') {
      steps {
        sh 'tar -czf calc-${BUILD_NUMBER}.tar.gz src requirement.txt'
        archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
      }
    }

    stage('Deploy') {
      when { branch 'main' }
      steps {
        sh 'mkdir -p /tmp/deploy && cp calc-*.tar.gz /tmp/deploy/ && ls -l /tmp/deploy'
      }
    }
  }

  post {
    success { echo "OK ${env.BRANCH_NAME} #${env.BUILD_NUMBER}" }
    unstable { echo "TESTS FAILED ${env.BRANCH_NAME} #${env.BUILD_NUMBER}" }
    failure { echo "FAILED ${env.BRANCH_NAME} #${env.BUILD_NUMBER}" }
    cleanup { cleanWs() }
  }
}