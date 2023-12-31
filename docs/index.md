# Jenkins Tutorial
My objective today is to demonstrate how to configure a CI/CD pipeline and deploy a containerized application with Jenkins using an AWS Cloud.
## What is Jenkins ?
Jenkins is a popular open source CI/CD tool. It is used to build and test your product continuously. To see this in action, I will walk you through a series of steps or stages.
## Tools and technology
Microservices in AWS, GitHub, Docker and Jenkins. 

## Table of contents
✔ Step 1: Start an EC2 Server with Enabled Ports. <br>
✔ Step 2: Connect to the Server via SSH. <br>
✔ Step 3: Update the Server. <br>
✔ Step 4: Install Java JDK and Jenkins Using Shell Script. <br>
✔ Step 5: Creating a Declarative Pipeline and Deploying our application.<br>
✔ Step 6: Full Automation with Webhooks. <br>
✔ Step 7: Further simplify our pipeline with "Pipeline Scripts from SCM" to make it more effective and easier to modify. <br>
### Step 1: Start an EC2 Server with Enabled Ports
### Step 2: Connect to the Server via SSH

```sh
sudo apt update
git clone https://github.com/GitHub-Username/name-of-repository.io
cd name-of repository
sudo apt install git docker.io -y
sudo usermod -aG docker ubuntu
sudo reboot
```

  - Reconnect using the SSH
  - Check if there are any ``` docker images ``` and if there's any
  - Build a docker Image using ``` docker build -t name-of-docker-image . ```
  - Check if Image exist ``` docker images ```
  - Check if Image is running ``` docker ps ```
  - Run Image ``` docker run -d -p 5000:5000 name-of-docker-image ```
  - Stop image ``` docker stop container's-id ```
### Step 3: Update the Server vis SSH
    sudo reboot
    sudo apt upgrade -y
### Step 4: Install Java JDK and Jenkins Using Shell Script
- Jenkins requires Java to run, Install Java JDK:

```
sudo apt install openjdk-11-jdk -y
```

- Add the Jenkins repository key and package source to apt:

```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
```

- Add Jenkins Repository:

```
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

NOTE!!!  If any error is encountered, use this script and replace the <PUBKEY> with the actual key values.

```
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv <PUBKEY>
gpg --export --armor <PUBKEY> | sudo apt-key add -
sudo apt-get update
```

- Update the Package List:

```
sudo apt update
```

- Install Jenkins:

```
sudo apt install jenkins
```

- Check the version:

```
jenkins --version
```

- Grant Jenkins access to build and run Docker images:

```
sudo usermod -aG docker jenkins
sudo reboot
```

- To get the Jenkins password <br>
  > To access Jenkins on the search bar put the instance:8080

    ```
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

### Step 5: Creating a Declarative Pipeline

Now, let's create a declarative pipeline in Jenkins:

1. Click on "New Item" and name it whatever, t-ex "cicd-pipeline".
2. Under the configuration settings, select the "GitHub project" and provide the GitHub repository URL:

    ```
    https://github.com/Github_Username/name-of-repository.git
    ```

    Set the name to "Segula-app"

3. Enable the "Build Triggers" option by ticking the box "GitHub hook trigger for GITScm pooling".
4. In the "Pipeline" section, choose "Pipeline Script" and add the following script:

    

        pipeline{
            agent any
            environment {
                DOCKERHUB_CREDENTIALS=credentials('dockerHub')
            }

            stages {
                stage('verify tools') {
                    steps {
                        sh '''
                            docker info
                            docker version
                            docker-compose version
                        '''
                    }
                }
                stage('gitclone') {
                    steps {
                        git url: "https://github.com/GitHub-Username/repository-name.git", branch: "main"
                    }
                }
                stage('Build') {
                    steps {
                        sh 'docker build -t dockerHub-username/dockerHub-image:latest .'
                    }
                }
                stage('Login') {
                    steps {
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    }
                }
                stage('Push') {
                    steps {
                        sh 'docker push DockerHub-username/dockerHub-image:latest'
                    }
                }
                stage("Deploy") {
                    steps {
                        echo "Deploying the container"
                        sh "docker-compose down"
                        // sh "docker-compose pull"  Pull the latest images
                        sh "docker-compose up -d" // Start the services in detached mode
                    }
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        } 
    

5. Add DockerHub credentials in the global credentials section with the ID as "dockerHub" along with the username and password.

6. Create a Docker-Compose file in your repository t.ex docker-compose.yml and paste the scripts below to enhance the deployment of Docker containers:

```sh
version: "3.3"
services:
web:
    image: dockerHub-username/dockerHub-image:latest
    ports:
    - "5000:5000"
```

7. Install Docker Compose by running the script below, before running the pipeline.

```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

```

Once everything is set up, you can manually trigger the pipeline by clicking the build button in the Jenkins interface. <br>
This will build successfully and the app is now deployed and ready to use. This is the benefits of automation and continuous integration.

### Step 6: Full Automation with Webhooks.
To achieve full automation, I will configure webhooks so that any changes pushed to the GitHub repository will trigger the Jenkins pipeline and initiate the deployment process automatically. To archieve that we.

- Go to your repository, settings, Webhooks, Add webhook.
- In the "Payload URL", paste the url ``` http://jenkins dashboard/github-webhook/ ``` and "Content type", select ```application/jason```.
- Update your code on your repository and see it update on the server.

### Step 7: Further simplify our pipeline with "Pipeline Scripts from SCM"

To make your pipeline more effective and easier to modify, you can use "Pipeline Script from SCM." This approach allows you to provide a GitHub repository link containing the Jenkinsfile. By adopting this method, any developer in your team with proper permissions can review and modify the Jenkinsfile.

- In your repository create a ```Jenkinsfile``` and paste the scripts
- In the Jenkins dashboard select ```Pipeline script from SCM```, Put the ```repository URL``` and ```Credential``` and the ```branch```. <br>

By leveraging webhooks and the "Pipeline Script from SCM" approach, your team can work collaboratively on the Jenkinsfile and make changes with ease, eliminating the need to access the Jenkins dashboard. 

## CI/CD Workflow Overview:

- Developer pushes code changes to the GitHub repository.
- Jenkins detects the code change event through webhook integration with GitHub.
- Jenkins starts the CI/CD pipeline, which includes steps to build, test, and package the application using Docker.
- Jenkins pushes the Docker image of the application to DockerHub.
- In a deployment stage, Jenkins or another tool (e.g., Kubernetes) can retrieve the Docker image from DockerHub and deploy it to a target environment, such as a server or cloud service.

Overall, this workflow enables a consistent and automated process for building, testing, and deploying applications, ensuring that changes are seamlessly propagated from source control to production.
