@Library('d13') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    d13Build()
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    d13Deployment()
                }
            }
        }
    }
}
