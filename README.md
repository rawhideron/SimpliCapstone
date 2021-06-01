I have chosen Project #2

I used ansible to run AWSEnvCreate.yml 
and create AWS ec2 instances.

Ansible executed JenkinsSetup.yml on ec2-3-93-32-82.compute-1.amazonaws.com to create a jenkins server.
I also setup Java (installJava.yml) as it is a Jenkins dependency. I setup maven (maven_install.yml)  but did not use it for this project.

DockerSetup/playbook.yml was an ansible script to setup docker on  ec2-54-83-228-224.compute-1.amazonaws.com

The pipeline I created is http://ec2-3-93-32-82.compute-1.amazonaws.com:8080/job/docker_git_trigger/
It consists of 7 steps:
- SCM checkout (from https://github.com/rawhideron/nginx which includes a webhook).
- Build docker image 
- Push Docker Image (https://hub.docker.com/repository/docker/rawhideron/my-app).
- Run the container
- perform curl test
- Remove container
- Remove image

Below is a copy of the pipeline script:

 stage('SCM checkout'){
        git url: 'https://github.com/rawhideron/nginx', branch: 'master'
    }
    
    stage('Build Docker Image'){
        sh "docker build -t rawhideron/my-app:3.1 ."
    }
    stage('Push Docker Image'){
       sh "docker push rawhideron/my-app:3.1"
    }
    stage('Run the container'){
        // sshAgent
        def dockerRun = "docker container run -p 8080:80 -d --name my-app  rawhideron/my-app:3.1"
        sshagent(['dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ubuntu@54.83.228.224 ${dockerRun}"
        }
    }
 
    stage('perform curl test'){
        sh 'curl http://ec2-54-83-228-224.compute-1.amazonaws.com:8080'
    
        
    }

    
   
    stage('Remove container'){
        def dockerContainerRemove = 'docker container rm my-app --force'
        sshagent(['dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ubuntu@54.83.228.224 ${dockerContainerRemove}"
        }
    }
    stage('Remove images'){
        def dockerImageRemove = 'docker image rm rawhideron/my-app:3.1 --force'
        sshagent(['dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ubuntu@54.83.228.224 ${dockerImageRemove}"
        }
    }  
    
}
