pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
               script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                      sh '''
                      chmod +x gradlew
                      ./gradlew sonarqube
                      '''
                    }

                timeout(5) {
                     def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
               }
            }
        }
        stage("building docker image and pushing it to nexus"){
           steps{
               script{

               withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    sh '''
                    sudo docker build -t 3.93.184.113:8083/springapp:${VERSION} .
                    sudo docker login -u admin -p $docker_password 3.93.184.113:8083
                    sudo docker push  3.93.184.113:8083/springapp:${VERSION}
                    sudo docker rmi 3.93.184.113:8083/springapp:${VERSION}  
                    sudo docker image prune -f      
                    '''
                  }
                }
              }
           }
        stage('Identifying misconfigs using datree in helm charts') {
            steps {
               script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=4f797888-557b-4ff1-9407-f3bd22d05d6b']){
                        sh 'sudo helm datree test myapp/' 
                        }
                    }
               }
            }
        }
    }
    post{
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "devboss7878@gmail.com";  
		    }
    }
}