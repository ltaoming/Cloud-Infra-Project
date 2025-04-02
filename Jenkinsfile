pipeline {
    agent none

    environment {
        scannerHome = tool 'taoming-sonar-tool'
        GCP_PROJECT = 'cloud-infra-project-455521'
        GCS_BUCKET = 'dataproc-staging-us-central1-909480457255-afl16jxa'
        JAVA_SRC_PATH = 'src/example/WordCount.java'
        CLUSTER_NAME = 'dataproc-cluster'
        REGION = 'us-central1'
        ZONE = 'us-central1-a'
        MASTER_NODE = 'dataproc-cluster-m'
    }

    stages {
        stage('SCM Checkout') {
            agent any
            steps {
                git branch: 'main', url: 'https://github.com/ltaoming/Cloud-Infra-Project'
            }
        }

        stage('Run Sonarqube') {
            agent any
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'taoming sonar installation') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Upload Java Source to GCS') {
            agent {
                docker {
                    image 'google/cloud-sdk:latest'
                }
            }
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $GCP_PROJECT
                        gsutil cp $JAVA_SRC_PATH gs://$GCS_BUCKET/src/
                    '''
                }
            }
        }

        stage('Compile and Run on Dataproc') {
            agent {
                docker {
                    image 'google/cloud-sdk:latest'
                }
            }
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $GCP_PROJECT

                        echo "SSH into Dataproc master and run Hadoop job..."

                        gcloud compute ssh $MASTER_NODE \
                          --zone=$ZONE \
                          --command="gsutil cp gs://$GCS_BUCKET/src/WordCount.java . && \
                                     mkdir -p classes && \
                                     javac -classpath \\$(hadoop classpath) -d classes WordCount.java && \
                                     jar -cvf WordCount.jar -C classes . && \
                                     gsutil -m rm -r gs://$GCS_BUCKET/output/ || true && \
                                     hadoop jar WordCount.jar WordCount gs://$GCS_BUCKET/input/wordcount-input.txt gs://$GCS_BUCKET/output/"
                    '''
                }
            }
        }
    }
}
