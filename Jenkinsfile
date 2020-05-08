pipeline {
    environment{
        registry = "shreya14/"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Get Maven'){
            agent{
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
        stage('Deliver') {
            agent any
            stages {
                stage('Building our image') {
                    steps{
                        script {
                            dockerImage = docker.build registry + ":$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy our image to dockerhub') {
                  steps{
                    script {
                      docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                      }
                    }
                  }
                }
                stage('Cleaning up') {
                  steps{
                      sh "docker rmi $registry:$BUILD_NUMBER"
                  }
                }
            }
        }
    }
}