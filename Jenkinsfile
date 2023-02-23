pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh "./gradlew sonarqube"
                    }

                    timeout(5) {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK'){
                            error("Pipeline aborted due to quality gate failure: ${qg.status}")
                        }
                    }
                }
            }
      
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-docker-pass', variable: 'docker-password')]) {
                        sh '''
                        docker build -t 35.244.94.13:8083/springapp:${VERSION} . 
                        echo "$docker-password"
                        docker login -u admin -p $docker-password 35.244.94.13:8083
                        docker push 35.244.94.13:8083/springapp:${VERSION}
                        docker rmi 35.244.94.13:8083/springapp:${VERSION}
                        '''
                    }
               
                }
            }
        }
    }
}