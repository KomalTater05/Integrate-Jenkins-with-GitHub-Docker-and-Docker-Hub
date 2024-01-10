# Integrate-Jenkins-with-GitHub-Docker-and-Docker-Hub

Deploy Jenkin as a docker container and configure it:

Write a Dockerfile  

Build the docker image

  Use jenkins/jenkins:lts  as the base image
  Use root user to install all the required packages
  Add apt -repo https://download.docker.com/linux/debian  to download the docker-ce edition
  Install docker and other required packages
  Finally add the jenkins user to docker group
Build the docker image

Run the docker container 

docker run -it -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped   <docker image> 

We should run Docker commands in our Jenkins container. So, what we need to do is to bind mount our container to our host machine daemon while we run the container using this argument: -v /var/run/docker.sock:/var/run/docker.sock
-v jenkins_home:/var/jenkins_home argument creates an explicit volume on our host machine. When we stop/restart/delete our container, we need to have our initial setup configuration intact.

Visit localhost 
http://localhost:8080
  use the initial password which is returned by the above commands and login
  customize jenkins by installing the required plugins such as docker and Kubernetes, git etc.
  JDK config and mvn config
  Jenkins container comes with an OpenJDK. To find it, we need to enter the container’s bash shell to get the JAVA_HOME path.
  $ docker exec -it <container_name/container_id>   /bin/bash
  echo $JAVA_HOME
  system configuration and go to global configurations
  add JDK installations by including path of the JAVA_HOME
  maven config
  similarly add the maven config

Jenkin project:

select Freestyle project:

select source Code Management (or SCM for short), select Git, add the remote Git repository URL of the project, and leave the branch field empty so any commit made to any branch triggers our entire Jenkins process:

Configure the GitHub project and add the project URL

Configure Build Triggers accordingly. (select Poll SCM, which checks whether we made changes and then rebuilds our project)

In the Build window, add two Invoke top-level Maven targets steps.
		

Build the project and make sure the build succeeds
console output should look a lot like the image below:


Make some changes in the source code and push it to the central repo. Make sure Jenkins will automatically build our Freestyle project(we don’t need to manually click Build Now)

Configure Jenkins to build the Docker image of our Java application and deploy that image to Docker Hub:
To perform this task, we need a few Jenkins plugins to be installed. In Manage Jenkins, select Manage Plugins under System Configurations, search and install the following plugins:

docker-build-step

CloudBees Docker Build and Publish

Create a Dockerfile with the below content
    FROM openjdk:8
    ADD target/java-jenkins-docker.jar java-jenkins-docker.jar
    ENTRYPOINT ["java", "-jar”, “java-jenkins-docker.jar"]
    EXPOSE 8080
    
Add the new file and then commit the changes to the GitHub repository. This will trigger a Jenkins post-commit build process as we configured.

Add our build steps to build and deploy a Java application’s Docker image:

 In the Jenkins build step set:
 
Repository name: <your repo> /<docker-image>

Example mathi/java-jenkins-docker

Provide Docker Hub access to Jenkin:

To give Jenkins access, we need to login to our Docker Hub account inside our Jenkins container through the command line, as shown below:

  $ docker exec -it <container_name/container_id> /bin/bash
  
Then inside the container, run the Docker login command:

$ docker login

Provide your credentials

Go back to your project and click Build Now, then navigate to the console output. The output should look, as shown in the image below.
The output should be something like this



# Steps Performed
- Build docker image of jenkins using: sudo docker build -t jenkins-image .
- Run the docker container : docker run -it -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped   <docker image>
- Visit localhost http://localhost:8080 to see jenkins up and running then configure the jenkins with maven and jdk path
- Configure and build a jenkins project
- Configure Jenkins to build the Docker image of our Java application and deploy that image to Docker Hub
    - Building the Dockerfile
    - Add our build steps to build and deploy a Java application’s Docker image
    - Provide Docker Hub access to Jenkin
# Screenshots of the Jenkin project and build status
![Screenshot (6)](https://github.com/KomalTater05/Integrate-Jenkins-with-GitHub-Docker-and-Docker-Hub/assets/155729023/7cf925cb-ac66-407f-88f3-7dbf80b2a940)
![Screenshot (7)](https://github.com/KomalTater05/Integrate-Jenkins-with-GitHub-Docker-and-Docker-Hub/assets/155729023/83105143-3ba2-4f03-b163-98d8ab1a0ee3)
# Screenshot of the DockerHub which shows the docker image
![Screenshot (8)](https://github.com/KomalTater05/Integrate-Jenkins-with-GitHub-Docker-and-Docker-Hub/assets/155729023/64c329ba-b498-40ee-8d63-2f750fc62450)



