@Library('nfq-library') _
pipeline {
  agent any

  environment {
        D13_BRANCH = 'd13_with_sidecar'
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
