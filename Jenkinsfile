pipeline {
    environment{
        registry = "shreya14/home"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    agent none
    stages {
        stage('Maven Build and test'){
            agent{
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            stages{
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
            }
        }
        stage('Deliver') {
            agent any
            stages {
                stage('Building image') {
                    steps{
                        script {
                            dockerImage = docker.build registry + ":$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploying image to dockerhub') {
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