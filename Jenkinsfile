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
        stage("building docker image and pushing it to DockerHub"){
           steps{
               script{

               withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    sh '''
                    docker build -t devboss7878/docker-hosted:${VERSION} .
                    docker login -u devboss7878 -p lex@luthor13
                    docker push  devboss7878/docker-hosted:${VERSION}
                    docker rmi devboss7878/docker-hosted:${VERSION}  
                    docker image prune -f      
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

        stage("Push the helm charts to Nexus"){
           steps{
               script{
                    dir('kubernetes/') {
                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                            sh '''
                                helmversion=$( sudo helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_password http://3.93.184.113:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                   }
                }
              }
           }
        
        stage('manual approval'){
            steps{
                script{
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "devboss7878@gmail.com";  
                        input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                        sh 'helm upgrade --install --set image.repository="devboss7878/docker-hosted" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        // sh 'kubectl get nodes'
                        }
                    }
               }
            }
        }
        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                         sh '''
                 
                         chmod +x healthcheck.sh
                         ./healthcheck.sh
                         '''

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