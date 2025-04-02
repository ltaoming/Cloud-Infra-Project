pipeline {
    agent any

    environment {
        scannerHome = tool 'taoming-sonar-tool'
    }

    stages {
        // 1. Pull code from GitHub
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ltaoming/Cloud-Infra-Project'
            }
        }

        // 2. Run SonarQube analysis
        stage('Run Sonarqube') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'taoming sonar installation') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        // 4. Compile Java and build JAR
        stage('Build JAR') {
            steps {
                sh '''
                    mkdir -p build
                    javac -d build src/WordCount.java
                    jar -cvf WordCount.jar -C build .
                '''
            }
        }

        // 5. Upload JAR to GCS bucket
        stage('Upload JAR to GCS') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project cloud-infra-project-455521
                        gsutil cp WordCount.jar gs://taoming-data-bucket/wordcount/WordCount.jar
                    '''
                }
            }
        }

        // 6. Run the MapReduce job on Dataproc
        stage('Run Hadoop Job on Dataproc') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project cloud-infra-project-455521

                        # Optional: remove output folder if it exists
                        gsutil -m rm -r gs://taoming-data-bucket/output/ || true

                        gcloud dataproc jobs submit hadoop \
                          --cluster=your-cluster-name \
                          --region=your-region \
                          --class=WordCount \
                          --jars=gs://taoming-data-bucket/wordcount/WordCount.jar \
                          -- gs://taoming-data-bucket/input/wordcount-input.txt gs://taoming-data-bucket/output/
                    '''
                }
            }
        }
    }
}
