pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                }
            }
            steps {
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
        stage("s3_upload") {
           agent any
           steps {
               // Upload deployment assets to S3
               sh "whoami"
               sh "aws s3 cp dist/add2vals s3://rtt-jenkins-bucket/training1/add2vals --region us-east-2 --acl public-read-write"
            }
        }
        stage("NexusPublish") {
           agent any
           steps {
               nexusArtifactUploader(
                   nexusVersion: 'nexus2',
                   protocol: 'http',
                   nexusUrl: 'ec2-18-218-233-46.us-east-2.compute.amazonaws.com:8081/nexus',
                   groupId: 'com.example',
                   version: version,
                   repository: 'RepositoryName',
                   credentialsId: 'NexusDefault',
                   artifacts: [
                       [artifactId: projectName,
                        classifier: '',
                        file: 'dist/add2vals',
                        type: 'binary']
                   ]
               )
           }
        }
    }
}
