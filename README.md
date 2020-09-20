# Problem_1

The processes i want to automate:

Code checkout
Run tests
Compile the code
Run Sonarqube analysis on the code
Create Docker image
Push the image to Docker Hub
Pull and run the image

First step, running up the services
Since one of the goals is to obtain the sonarqube report of our project, we should be able to access sonarqube from the jenkins service. Docker compose is a best choice to run services working together.

Paths of docker files of the containers are specified at context attribute in the docker-compose file. Content of these files as follows.

sonarqube/Dockerfile

FROM sonarqube:6.7-alpine
jenkins/Dockerfile

FROM jenkins:2.60.3
If we run the following command in the same directory as the docker-compose.yml file, the Sonarqube and Jenkins containers will up and run.

docker-compose -f docker-compose.yml up --build


GitHub configuration
We’ll define a service on Github to call the Jenkins Github webhook because we want to trigger the pipeline. To do this go to Settings -> Integrations & services. The Jenkins Github plugin should be shown.

After this, we should add a new service by typing the URL of the dockerized Jenkins container along with the /github-webhook/ path.

The next step is that create an SSH key for a Jenkins user and define it as Deploy keys on our GitHub repository.
If everything goes well, the following connection request should return with a success.

ssh git@github.com
PTY allocation request failed on channel 0
Hi <your github username>/<repository name>! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.


Jenkins configuration
i have configured Jenkins in the docker compose file to run on port 8080 therefore if we visit http://localhost:8080 we will be greeted.

We need the admin password to proceed to installation. It’s stored in the /var/jenkins_home/secrets/initialAdminPassword directory and also It’s written as output on the console when Jenkins starts.

jenkins      | *************************************************************
jenkins      |
jenkins      | Jenkins initial setup is required. An admin user has been created and a password generated.
jenkins      | Please use the following password to proceed to installation:
jenkins      |
jenkins      | 45638c79cecd4f43962da2933980197e
jenkins      |
jenkins      | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
jenkins      |
jenkins      | *************************************************************
To access the password from the container.

docker exec -it jenkins sh
/ $ cat /var/jenkins_home/secrets/initialAdminPassword
After entering the password, we will download recommended plugins and define an admin user.

After clicking Save and Finish and Start using Jenkins buttons, we should be seeing the Jenkins homepage. One of the seven goals listed above is that we must have the ability to build an image in the Jenkins being dockerized. Take a look at the volume definitions of the Jenkins service in the compose file.

- /var/run/docker.sock:/var/run/docker.sock
The purpose is to communicate between the Docker Daemon and the Docker Client(we will install it on Jenkins) over the socket. Like the docker client, we also need Maven to compile the application. For the installation of these tools, we need to perform the Maven and Docker Client configurations under Manage Jenkins -> Global Tool Configuration menu.

We have added the Maven and Docker installers and have checked the Install automatically checkbox. These tools are installed by Jenkins when our script(Jenkins file) first runs. We give myMaven and myDocker names to the tools. We will access these tools with this names in the script file.

Since we will perform some operations such as checkout codebase and pushing an image to Docker Hub, we need to define the Docker Hub Credentials. Keep in mind that if we are using a private repo, we must define Github credentials. These definitions are performed under Jenkins Home Page -> Credentials -> Global credentials (unrestricted) -> Add Credentials menu.

we select GitHub hook trigger for GITScm pooling options for automatic run of the pipeline by Github hook call.
Also in the Pipeline section, we select the Pipeline script from SCM as Definition, define the GitHub repository and the branch name, and specify the script location ([https://github.com/vineet68sharma/CI-CD-Docker/blob/master/Jenkinsfile)).

Create a repository in docker hub so the built image is pushed to docker registry with its credentials (need to be filled in jenkinsFile)

After that, when a push is done to the remote repository or when you manually trigger the pipeline by Build Now option, the steps described in Jenkins file will be executed.

After that, when a Deployment is done to the remote repository or when you manually trigger the pipeline by Build Now
After Deployment is done and Container is running , You can check it by URL and Port number on which the service is running

http://ec2-50-16-10-93.compute-1.amazonaws.com:8090/
withCredentials provided by Jenkins Credentials Binding Plugin and bind credentials to variables. We passed dockerHubAccount value with credentialsId parameter. Remember that, dockerHubAccount value is Docker Hub credentials ID we have defined it under Jenkins Home Page -> Credentials -> Global credentials (unrestricted) -> Add Credentials menu. In this way, we access to the username and password information of the account for login.


