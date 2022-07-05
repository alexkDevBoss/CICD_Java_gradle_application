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
                    sudo docker login -u admin -p lex@luthor13 3.93.184.113:8083
                    sudo docker push  3.93.184.113:8083/springapp:${VERSION}
                    sudo docker rmi 3.93.184.113:8083/springapp:${VERSION}  
                    sudo docker image prune -f      
                    '''
                  }
                }
              }
           }
    }
    post{
        always{
            echo "SUCCESS"
        }
    }
}