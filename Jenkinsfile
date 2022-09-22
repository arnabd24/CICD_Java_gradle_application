pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                      withSonarQubeEnv(credentialsId: 'sonar') {
                         sh 'chmod +x gradlew'
                         sh './gradlew sonarqube'
                      }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage("Docker build and Docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                              docker build -t 13.232.122.149:8083/springapp:${VERSION} .
                              docker login -u admin -p ${docker_password} 13.232.122.149:8083
                              docker push 13.232.122.149:8083/springapp:${VERSION}
                              docker rmi 13.232.122.149:8083/springapp:${VERSION}
                        ''' 
                        }

                    
                }
            }
        }
     }
}