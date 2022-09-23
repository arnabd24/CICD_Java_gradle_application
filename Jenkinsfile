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
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        dir('kubernetes/') {
                        sh '''
                           helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                           tar -czvf myapp-${helmversion}.tgz myapp/
                           curl -u admin:$docker_password http://13.232.122.149:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                        ''' 
                        }
                    }

                    
                }
            }
        }
        stage{
            steps{
                script{
                    kubeconfig(credentialsId: 'kubeconfig', serverUrl: 'https://172.31.43.91:6443'){
                        dir ("kubernetes/"){
                            sh 'helm list'
                            sh 'helm upgrade --install --set image.repository="172.31.41.85:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                             }
                    } 
                }
            }
        }
        
     }
     
}