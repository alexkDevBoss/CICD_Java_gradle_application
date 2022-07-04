pipeline{
    agent any
    stages{
        stage("sonar quality check"){
            agent{
                sudo docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
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