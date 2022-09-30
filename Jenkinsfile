pipeline {

   agent any 

   stages {

     stage("Git checkout commit"){
        step{
             git credentialsId: 'GIT_HUB_ID', url: 'http://github.com/'
        }
     }

     stage("Gradle build and compile"){
        step{
             sh './gradlew build'
        }
     }
 

    stage("Docker build"){
        step{
            sh 'docker version'
            sh 'docker build -t imageName .'
            sh 'docker images'
            sh 'docker tag username/repoName imageName:latest'
            sh 'docker '
        }
     }

        stage("Docker login"){
        step{
            withCredentials([String(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]){
                sh 'docker login -u  your dockerhub username -p $PASSWORD'
            }
        }
     }

      stage("Push image to DockerHub"){
        step{
            sh 'docker push username/repoName imageName:latest'
        }
     }

       stage("Kubernetes Deployment"){
        step{
            sh 'kubectl apply -f deploymentName.yml'
        }
     }

     
   }

}