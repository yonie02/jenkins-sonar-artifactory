pipeline {
    agent any
    environment {
    CI = true
    ARTIFACTORY_ACCESS_TOKEN = credentials('artifactory-access-token')
    JFROG_PASSWORD  = credentials('jfrog-password')
  }
    stages {
        stage('Dependecies') {
            steps {
                sh 'java --version'
            }
        }
        stage('Build Maven Job'){
           steps {
              sh 'mvn clean install'
           }
        }
       stage('Upload Binaries to Jfrog Artifactory') {
        steps {
        sh 'jf rt upload --url http://lab.cloudsheger.com:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} target/demo-0.0.1-SNAPSHOT.jar java-web-app/'
          }
        }
      stage('Building Docker Image'){
          steps{
              sh '''
              docker build -t localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER --pull=true .
              docker images
              '''
          }
      }
      stage('Image Scanning Trivy'){
            steps{
               sh 'trivy image localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER > $WORKSPACE/trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt'
            }
     }
     stage('Uploading Image Scan to Jrog Artifactory'){
         steps{
          sh 'jf rt upload --url http://lab.cloudsheger.com:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt trivy-scan-files/'           
         }
     }
     stage('Pushing Docker Image into Jfrog'){
         steps{
             sh '''
             docker login java-web-app-docker.jfrog.io -u admin -p ${JFROG_PASSWORD}
             docker push localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER
             '''
        }
     }
     stage('Cleaning up DockerImage'){
            steps{
                sh 'docker rmi localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER'
           }
       }
  }
}
