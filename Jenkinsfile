pipeline {
    agent any 
    
    environment {
        scannerHome = tool 'taoming-sonar-tool'
    }

    stages { 
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ltaoming/Cloud-Infra-Project'
            }
        }

        stage('Run Sonarqube') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'taoming sonar installation') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // This will wait for the SonarQube analysis to be available and check the Quality Gate
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Authenticate GCloud') {
            steps {
                withCredentials([file(credentialsId: 'gcloud-service-account-key', variable: 'GCLOUD_KEY')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GCLOUD_KEY
                        gcloud config set project <your-gcp-project-id>
                    '''
                }
            }
        }

        stage('Connect to Dataproc') {
            steps {
                sh '''
                    gcloud dataproc clusters describe dataproc-cluster --region=<your-region>
                    # Example: Run a simple job or SSH into a node
                    # gcloud dataproc jobs submit spark --cluster=dataproc-cluster --region=<your-region> --class=org.apache.spark.examples.SparkPi --jars=file:///usr/lib/spark/examples/jars/spark-examples.jar -- 100
                '''
            }
        }

        stage('Upload and Compile WordCount') {
            steps {
                script {
                    def REGION = '<your-region>'
                    def CLUSTER = 'dataproc-cluster'
                    def PROJECT = '<your-gcp-project-id>'
                    def MASTER_NODE = sh(
                        script: "gcloud dataproc clusters describe ${CLUSTER} --region=${REGION} --project=${PROJECT} --format='value(config.masterConfig.instanceNames[0])'",
                        returnStdout: true
                    ).trim()

                    def ZONE = sh(
                        script: "gcloud compute instances list --filter='name=${MASTER_NODE}' --format='value(zone)'",
                        returnStdout: true
                    ).trim()

                    sh """
                        # Copy the WordCount.java file to master node
                        gcloud compute scp src/example/WordCount.java ${MASTER_NODE}:~/ --zone=${ZONE}
                        
                        # Compile the Hadoop job on the master node
                        gcloud compute ssh ${MASTER_NODE} --zone=${ZONE} --command='
                            mkdir -p wordcount_classes &&
                            javac -classpath $(hadoop classpath) -d wordcount_classes WordCount.java &&
                            jar -cvf wordcount.jar -C wordcount_classes/ .
                        '
                    """
                }
            }
        }
        
        stage('Run Hadoop Job') {
            steps {
                script {
                    def REGION = '<your-region>'
                    def CLUSTER = 'dataproc-cluster'
                    def PROJECT = '<your-gcp-project-id>'
                    def GCS_INPUT = 'gs://dataproc-staging-us-central1-909480457255-afl16jxa/input'
                    def GCS_OUTPUT = 'gs://dataproc-staging-us-central1-909480457255-afl16jxa/output-${BUILD_NUMBER}'

                    def MASTER_NODE = sh(
                        script: "gcloud dataproc clusters describe ${CLUSTER} --region=${REGION} --project=${PROJECT} --format='value(config.masterConfig.instanceNames[0])'",
                        returnStdout: true
                    ).trim()

                    def ZONE = sh(
                        script: "gcloud compute instances list --filter='name=${MASTER_NODE}' --format='value(zone)'",
                        returnStdout: true
                    ).trim()

                    sh """
                        gcloud compute ssh ${MASTER_NODE} --zone=${ZONE} --command='
                            hadoop fs -rm -r ${GCS_OUTPUT} || true &&
                            hadoop jar wordcount.jar WordCount ${GCS_INPUT} ${GCS_OUTPUT}
                        '
                    """
                }
            }
        }
    }
}
