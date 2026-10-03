pipeline {
    agent {
        kubernetes {
            label 'maven-agent'
            defaultContainer 'maven'
        }
    }
    // add first stage Test with command "./mvnw verify"
    stages {
        stage('Test') {
            steps {
                container('maven') {
                    sh './mvnw verify'
                }
            }
        }
    }
}