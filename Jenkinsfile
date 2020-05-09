pipeline {
    environment{
        registry = "shreya14/calculator"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    agent none
    stages {
        stage('Maven Build + Test'){
            agent{
                docker {
                    image 'maven:3.6.3-jdk-11'
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
        stage('Deploy') {
            agent any
            steps {
                script {
          step([$class: "RundeckNotifier",
          rundeckInstance: "rundeck",
          options: """
            BUILD_VERSION=$BUILD_NUMBER
          """,
          jobId: "1aa9705b-10a5-4ca9-910a-6433d9b96f80"])
        }
      }
    }
    }
}
