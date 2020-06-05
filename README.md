# jenkins

Follow below steps to create a customized jenkins image.

1. docker build --rm -f jenkins/dockerFileDocker.yml -t chiragjenkinsdocker:v1 jenkins

    Copy the above dockerFile file into a folder and outside the folder run above command. For keeping it simple just create a new folder that is having this file only.

    Here dockerFileDocker.yml is the name of the file.
    
    chiragjenkinsdocker:v1 is the name of the Image with any tag. In my case chiragjenkinsdocker is the image name and v1 is the image tag.
    
    If any tag is not provided, "latest" tag is considered.
    
2. Once Image is created just run the below command to run the image.
  
    docker run -d --name jenkinscontainer -p 8090:8080 -v /var/run/docker.sock:/var/run/docker.sock chiragjenkinsdocker:v1
    
    Explaination : jenkinscontainer is the dedicated name of the container. You can obviously change it according to your choice.
    
                   8090:8080 mapping of 8080 container port with host machine 8090 port. 
                    
                   /var/run/docker.sock:/var/run/docker.sock  this is to mount the Docker socket inside out.
                   
                   chiragjenkinsdocker:v1 this is to tell docker which image is to run.
                   
 3. There might be a case that jenkins could not find docker sock. For that simply give the execute access.
 
        sudo chmod 666 /var/run/docker.sock
        
     
 *********************************************************Pipeline******************************************************************
 
 PRE-REQUISITES:
 - 
 + Install Plugins those are related to GIT + DOCKER + DOCKER PIPELINE + or any other as per your need.
 
 1. Add github/dockerhub credentials first with a dedicated id for both.(These ids we will use in pipeline script).
 
 2. You are all set to write a Pipeline script.
 
 3. Create a new Pipeline Job and write the Pipeline Script below for reference.
 
 
        
        
        pipeline {
              environment {
                registry = "chiraggupta95/stockkeeper-backend"
                registryCredential = 'dockerhub'
                dockerImage = ''
              }
              agent any
              stages {
                stage('Cloning Git') {
                  steps {
                    git url: 'https://gitlab.com/chiraggupta95apr/stockkeeper.git',
                        branch: "$GIT_BRANCH",
                        credentialsId: '698d3bfd-8f7d-4a6d-a3f0-55a5b2dc9b60'
                  }
                }
                stage('Build Maven') { 
                        steps {
                            sh 'mvn -B -DskipTests clean package' 
                        }
                    }
                stage('Building image') {
                  steps{
                    script {
                      dockerImage = docker.build registry + ":$RELEASEVERSION"
                    }
                  }
                }
                stage('Deploy Image') {
                  steps{
                    script {
                      docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                      }
                    }
                  }
                }
                stage('Remove Unused docker image') {
                  steps{
                    sh "docker rmi $registry:$RELEASEVERSION"
                  }
                }
              }
}

EXPLANATION:
-

 registryCredential = 'dockerhub'    Here dockerhub is the credential id that we created in Step 1.
 
 credentialsId: '698d3bfd-8f7d-4a6d-a3f0-55a5b2dc9b60' This is an example if you skip putting any id in Step 1. Jenkins auto-generate an                                                        id incase you miss it.
 $GIT_BRANCH This is a choice parameter that will be helpful in selecting the branch you want to build.
 
 $RELEASEVERSION This is a choice parameter that will be helpful in Tagging the Image with a Release Version.
 
 
 4. Save the Pipeline and Build the job after selecting build parameters.
 

Hope this helps.
