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
                    withCredentials([string(credentialsId: 'nexus-docker-pass', variable: 'docker_password')]) {
                        sh '''
                        docker build -t 35.189.3.147:8083/springapp:${VERSION} . 
                        docker login -u admin -p $docker_password 35.189.3.147:8083
                        docker push 35.189.3.147:8083/springapp:${VERSION}
                        docker rmi 35.189.3.147:8083/springapp:${VERSION}
                        '''
                    }
               
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "lucas95wang@gmail.com";  
		}
	}
}