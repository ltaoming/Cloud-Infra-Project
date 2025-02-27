pipeline {
    agent any 

    stages { 
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ltaoming/Cloud-Infra-Project'
            }
        }

        // Compile Java code before running SonarQube
        stage('Build') {
            steps {
                sh './gradlew clean build' 
            }
        }

        // Run SonarQube analysis
        stage('Run Sonarqube') {
            environment {
                scannerHome = tool 'SonarQubeScanner';
            }
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'SonarQube') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=SonarQube-ltaoming \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=build/classes/java/main \
                        -Dsonar.host.url=http://your-sonarqube-server
                    """
                }
            }
        }
    }
}
