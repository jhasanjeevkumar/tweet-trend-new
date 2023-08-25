pipeline {
    agent {
        node{
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    }

    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('sonar-analysis') {
            environment {
                scannerHome = tool 'sjha01-sonar-scanner'
            }
            steps{
                withSonarQubeEnv('sjha01-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }

        }
    }
}
