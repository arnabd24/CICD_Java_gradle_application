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
                              docker build -t 172.31.41.85:8083/springapp:${VERSION} .
                              docker login -u admin -p ${docker_password} 172.31.41.85:8083
                              docker push 172.31.41.85:8083/springapp:${VERSION}
                              docker rmi 172.31.41.85:8083/springapp:${VERSION}
                        ''' 
                        }

                    
                }
            }
        }
        stage('Identify misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=ae2a076c-9f61-485d-a5b2-13b71109e19a']) {
                            sh 'helm datree test myapp/'
                        }
                        
                    }
                }
            }
        }
        
     }
     post {
		   always {
			    mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "dasarn94@gmail.com";  
		       }
	      }
}