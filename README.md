# Automated Build and Test with Jenkins on EC2 Instance
This project aims to automate the build and testing processes of software development using Jenkins, and the Git Flow branching model for efficient code management

## Project Setup
### Tools:
- EC2 instance on AWS with docker, git and git flow installed
- Jenkins container
- GitHub account

## Installation and Configuration Guide

 ### 1. Create EC2 Instance on AWS:
 - Launch an EC2 instance on AWS
 - Edit security group settings:
   Add inbound rule to the security group for port 8080
   - type: custom TCP
   - port :8080
   - source: 0.0.0.0/0
 
 ### 2. Prepare GitHub Repository:
 - Create a new GitHub repository or select an existing one
 - Initialize the repository with a README.md
 - Ensure the repository contains at least two branches:
   - develop: for ongoing development
   - master (or main): for stable releases
 - Apply the Git Flow model
   
 ### 3. Install Git and Git Flow on EC2
 ```
#Installing Git

$ apt install git
$ git --version


#Clone the repo

$ git clone <repo_url>
$ ls
$ cd <path/to/repo>
```
```
#Installing Git Flow

$ git clone --recursive https://github.com/petervanderdoes/gitflow-avh.git
$ cd  gitflow-avh /contrib/
$ chmod +x gitflow-installer.sh
$ ./gitflow-installer.sh install stable
$ apt update -y
```
```
#Initialize Git & Git Flow

$ git init
$ git flow init  # Keep the defaults
$ git branch  # Check the assigned branch
```


 ### 4. Create Jenkinsfile and Push to GitHub
```
$ vim Jenkinsfile

#Write the following lines in the file

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}


#Save and Exit
```

```
$ git add Jenkinsfile
$ git commit -m "Adding Jenkinsfile"
$ git push <repo_url> develop
```

 ### 5. Install Jenkins on EC2
```
#Switch to root

$ apt update -y
$ apt install docker.io -y
$ docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

**Accessing Jenkins:**
- Go to web browser and write : http://<ec2-public-ip>:8080
- Enter the container
  ```
  $ docker exec -it <container_id> bash
  ```
- Run the following command inside your container to get Jenkins password
  ```
  $ cat /var/jenkins_home/secrets/initialAdminPassword
  ```
- Copy password to Jenkins tab and sign in

### 6. Configuring Jenkins:
- Install necessary plugins in Jenkins, if not already available, such as Git and Pipeline
- Connect Jenkins to GitHub
  - Go to "Manage Jenkins" > "Manage Plugins" > "Available" and install "GitHub Integration Plugin"
  - Set up credentials in Jenkins for GitHub (username and token).
- Create a new pipeline job
  - Select "New Item", name your pipeline (e.g., "GitHub Pipeline"), and choose "Pipeline" as the type
  - In the pipeline configuration, select "Pipeline script from SCM" and choose "Git" as the SCM
  - Enter the repository URL and credentials
  - Specify the branch to build (e.g., */develop)
  - In Build Triggers, Check "GitHub hook trigger for GITScm polling"
  - Save

### 7. Implement Webhooks for Continuous Integration:
Set up webhooks in GitHub to trigger Jenkins builds on push events:
- In GitHub, go to your repository settings and select "Webhooks"
- Add a new webhook:
  - Payload URL: http://<jenkins_url>:8080/github-webhook/
  - Content type: application/json
  - Select "Just the push event"
  - Ensure the webhook is active

With the webhook, Jenkins will trigger a new build every time changes are pushed to the connected branch.

### 8. Testing and Validation:
- Push a change to the develop branch and verify Jenkins triggers a build
- Check the Jenkins dashboard for build status and output

 
