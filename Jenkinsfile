@Library('nfq-library') _
pipeline {
  agent any

  environment {
        D13_BRANCH     = 'd13_with_sidecar'
        D13_PROJECT    = 'test-d13-jenkins'
        D13_REPOSITORY = 'test-d13-jenkins'
  }

  options {
    ansiColor('xterm')
    timestamps()
    disableConcurrentBuilds(abortPrevious: true)
  }

  stages {
    stage('Build') { steps { script { d13Build() } } }
    stage('Deployment') { steps { script { d13Deployment() } } }
  }

  post { always { d13Clean(); cleanWs() } }
}
