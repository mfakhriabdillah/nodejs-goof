pipeline {
    agent none
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockercred')
        // SNYK_CREDENTIALS = credentials('SnykToken')
    }
    stages {
        stage('Secret Scanning using Trufflehog') {
            agent {
                docker {
                    image 'trufflesecurity/trufflehog:latest'
                    args '-u root --entrypoint='
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'trufflehog filesystem . --exclude-paths trufflehog-excluded-paths.txt --fail > trufflehog-scan-result.txt'
                }
                sh 'cat trufflehog-scan-result.txt'
                archiveArtifacts artifacts: 'trufflehog-scan-result.txt'
            }        
        stage('Build') {
            agent {
                docker {
                    image 'node:lts-buster-slim'
                }   
            }
            steps{
                sh 'npm install'
            }
        }
        }
        stage('Test') {
            agent{
                docker {
                    image 'node:lts-buster-slim'
                }
            }
            steps{
                sh 'echo "Test Skipped"'
            }
        }
        stage('Build Docker Images') {
            agent{
                docker {
                    image 'docker:dind'
                    args '--user root --network host -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps{
                sh 'docker build -t fakhriabdillah/nodejsgoof:0.${BUILD_NUMBER} .'
            }
        }
        stage('Push Images to Docker Registry') {
            agent{
                docker {
                    image 'docker:dind'
                    args '--user root --network host -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push fakhriabdillah/nodejsgoof:0.${BUILD_NUMBER}'
            }
        }
    }
}