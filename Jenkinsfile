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

        stage('Quality Gate Check') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Quality Gate Failed: ${qg.status}"
                    } else {
                        echo "No critical issues found. Proceeding to Hadoop execution."
                    }
                }
            }
        }

        stage('Run Hadoop Job on Dataproc') {
            steps {
                script {
                    sh """
                    gcloud dataproc jobs submit hadoop \
                        --cluster=${HADOOP_CLUSTER} \
                        --region=${REGION} \
                        --class=org.apache.hadoop.examples.WordCount \
                        --jars=file:///usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar \
                        -- input-bucket/input.txt output-bucket/output
                    """
                }
            }
        }

        stage('Fetch Hadoop Results') {
            steps {
                script {
                    sh """
                    gsutil cp gs://output-bucket/output/part-r-00000 ./output.txt
                    """
                    echo "Hadoop job completed successfully. Results saved."
                }
            }
        }
    }
}
