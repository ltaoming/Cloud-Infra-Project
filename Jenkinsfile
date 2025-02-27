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
                scannerHome = tool 'SonarQubeScanner';
            }
            steps {
              withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
        }
    }
}
