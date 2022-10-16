pipeline{
    agent any
    stages{
        stage("Sonarqube Testing"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    sh 'chmod +x gradlew'
                    sh './gradlew sonarqube -Dsonar.host.url=http://172.31.40.174:9000 -Dsonar.login=sqa_3b2289a7cb02ba863957ed1e76ed6020a8427cf8'
                }
            }
        }
    }
}
