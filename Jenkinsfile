pipeline {
    agent any
    environment {
    CI = true
    ARTIFACTORY_ACCESS_TOKEN = credentials('artifactory-access-token')
    JFROG_PASSWORD  = credentials('jfrog-password')
    DOCKER_IMAGE = 'prod'
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
              docker build -t localhost:8082/docker-repo-key/demoapp:${DOCKER_IMAGE} --pull=true .
              docker images
              '''
          }
      }
      stage('Image Scanning Trivy'){
            steps{
            securityScan()
          }
        }
     stage('Uploading Image Scan to Jrog Artifactory'){
         steps{
          sh 'jf rt upload --url http://lab.cloudsheger.com:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} trivy-image-scan/trivy-image-scan-${DOCKER_IMAGE}.txt trivy-scan-files/'           
         }
     }
     stage('Pushing Docker Image into Jfrog'){
         steps{
             sh '''
             docker login java-web-app-docker.jfrog.io -u admin -p ${JFROG_PASSWORD}
             docker push localhost:8082/java-web-app-docker/demoapp:${DOCKER_IMAGE}
             '''
        }
     }
     stage('Cleaning up DockerImage'){
            steps{
                sh 'docker rmi localhost:8082/java-web-app-docker/demoapp:${DOCKER_IMAGE}'
           }
       }
    // skip a stage while creating the pipeline
      stage("A stage to be skipped") {
         when {
            expression { false }  //skip this stage
         }
         steps {
            echo 'This step will never be run'
         }
      }
      
      // Execute when branch = 'master'
      stage("BASIC WHEN - Branch") {
         when {
            branch 'master'
	 }
         steps {
            echo 'BASIC WHEN - Master Branch!'
         }
      }   
  }
}

def securityScan() {
sh '''
mkdir -p trivy-image-scan
cd trivy-image-scan && touch trivy-image-scan-${DOCKER_IMAGE}.txt
trivy image localhost:8082/docker-repo-key/demoapp:${DOCKER_IMAGE} > $WORKSPACE/trivy-image-scan/trivy-image-scan-${DOCKER_IMAGE}.txt
whoami 
  '''
}

//sh 'trivy image localhost:8082/docker-repo-key/demoapp:${DOCKER_IMAGE} > $WORKSPACE/trivy-image-scan/trivy-image-scan-${DOCKER_IMAGE}.txt'

 