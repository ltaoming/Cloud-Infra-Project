pipeline {
    agent any 
    
    stages { 
        stage('SCM Checkout') {
            steps{
           git branch: 'main', url: 'https://github.com/ltaoming/Cloud-Infra-Project'
            }
        }
        // run sonarqube test
        stage('Run Sonarqube') {
            environment {
                scannerHome = tool 'taoming-sonar-tool';
            }
            steps {
              withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'taoming sonar installation') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
        }
    }
}
