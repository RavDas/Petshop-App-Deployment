We will be deploying our application in two ways, one using Docker Container and the other using K8S cluster.


Step 1 - Create an Ubuntu(22.04) T2 Large Instance 

Step 2 - Install Jenkins, Docker and Trivy

Step 3 - Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check

Step 4 - Configure Sonar Server in Manage Jenkins

Step 5 - Install OWASP Dependency Check Plugins

Step 6 - Docker plugin and credential Setup

Step 7 - Adding Ansible Repository in Ubuntu and install Ansible

Step 8 - Kuberenetes Setup

Step 9 - Master-slave Setup for Ansible and Kubernetes


Now let's start the Deployment process.


### Step 1 -  Create an Ubuntu(22.04) T2 Large Instance - 30GB Storage (To install Jenkins with plugins, SonarQube, Trivy)

Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable ssh (22), HTTP(80) and HTTPS(443) traffic by opening ports in the instance settings in the Security Group.

![image](https://github.com/user-attachments/assets/a61ddcd3-8913-42a2-9bf9-97c3948da5fa)

Now click on "Connect" button on "Instance" Dashboard. Then ckick on "SSH Client".

Here you need to have the created key-pair(.pem file) in the file directory of the your local machine that you launch your console inorder to ssh into EC2 instance which is attached with the same key-pair(.pem file).

![image](https://github.com/user-attachments/assets/85ac7c7d-67c1-4c7a-8d2c-645adfc22cde)

![command](https://github.com/user-attachments/assets/1c9f7d39-8332-482a-a80c-8af558be8cc5)


**OR**

ssh into the EC2 instance using GUI of the MobaXtreme termminal like below (Here you can simply attach the key-pair in "Use private key" section).

![image](https://github.com/user-attachments/assets/2ae5a416-c1fc-4381-a04c-84cecfee8095)


### Create your repository, and push those into your GitHub repository

1. Clone the repo:

```bash
git clone https://github.com/RavDas/Petshop-App-Deployment.git
```

2. Initialize Git Repository

```bash
git init
```

3. Add Files to Git:

```bash
# Stage all files for the first commit:

git add .
```

4. Commit Files:

```bash
# Commit the staged files with a commit message:

git commit -m "Initial commit"
```

5. Change the remote repo

```
git remote add new-origin https://github.com/RavDas/Petshop-App-Deployment.git
```
replace with your GitHub repo

6. Push to GitHub:

```bash
# Push the local repository to GitHub:

git push -u origin main
```
### Step 2 — Install Jenkins, Docker and Trivy

#### 2A - Install Java runtime, Jenkins


Connect to your console, and enter these commands to Install Jenkins

```
vi jenkins.sh
```

```
#!/bin/bash
sudo apt update -y
sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Then run,

```
sudo chmod 666 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

Since Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080(default port) and 9000 for Jenkins and SonarQube to work.

But for this Application case, we are running Jenkins on another port(8090) since we use port 8080 to run Apache Maven which is used to build the source code. So change the port to 8090 using the below commands.

```
sudo systemctl stop jenkins
sudo systemctl status jenkins
cd /etc/default
sudo vi jenkins   #change port "HTTP_PORT=8090" and save and exit
```

![http](https://github.com/user-attachments/assets/f8a882c9-263b-417c-b3b5-469d90e04952)

```
cd /lib/systemd/system
sudo vi jenkins.service  #change Environments="JENKINS_PORT=8090", save and exit
```

![jenkin prot](https://github.com/user-attachments/assets/e3e35f88-2ba4-4c56-9749-a7cbb10a40a3)


Reload Jenkins

```
sudo systemctl daemon-reload
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

Now get your Public IP Address of the EC2 instance created.

```
<EC2 Public IP Address:8090>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Unlock Jenkins using an administrative password using above command and install the suggested plugins.

![image](https://github.com/user-attachments/assets/b335e4ad-4058-4f64-a73e-00b72fae0ac7)

Jenkins will now get installed and install all the libraries.

![image](https://github.com/user-attachments/assets/345fa79e-d612-4cbe-b49f-42290e5bfb31)

Create a user click on save and continue.

![image](https://github.com/user-attachments/assets/fa14867e-e20b-49dd-892e-a89e1cd7c9d5)

Jenkins Getting Started Screen.

![image](https://github.com/user-attachments/assets/548294e7-4a48-48b2-88a7-8b2b91bbffc4)


#### 2B - Installation of Docker on the VM

```
# 1. Update the package list and install necessary packages
sudo apt-get update 
sudo apt-get install ca-certificates curl

# 2. Download and add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings 
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc 
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 3. Add Docker repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Update package index
sudo apt-get update

# 5. Install Docker packages
sudo apt-get install docker-ce docker-ce-cli containerd.io -y

# 6. Add user to Docker group
sudo usermod -aG docker $USER

# 7. Apply group changes (log out and back in or use the following command)
newgrp docker

# 8. Grant permission to Docker socket to push into the Docker Hub (optional, for convenience)
sudo chmod 666 /var/run/docker.sock
```

Following these steps, you should have successfully installed Docker on your Ubuntu system. You can now start using Docker to containerize and manage your applications.

Follow this official document if you find any errors: Link: https://docs.docker.com/engine/install/ubuntu/


After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).

-run -> docker run -d –name sonar -p 9000:9000 sonarqube:lts-comminity

![5](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/6aae6964-12db-475d-8f58-39e3369d00a2)

-access using <public_ip:9000>

Now our SonarQube is up and running,

username: admin and password:admin

![6](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/3742eb3d-ed0c-457d-ac2e-4b721b5c9c01)

Enter username and password, click on login and change password

![7](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/bb900408-1f6a-42e0-9635-3ec67d6fc6da)

Update New password, This is Sonar Dashboard.

![8](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/6171a1fa-bfc4-4e31-9d11-c4054bb396de)








## Run on Application Server
Running JPetStore sample under Tomcat (using the [cargo-maven2-plugin](https://codehaus-cargo.github.io/cargo/Maven2+plugin.html)).

- Clone this repository

  ```
  $ git clone https://github.com/mybatis/jpetstore-6.git
  ```

- Build war file

  ```
  $ cd jpetstore-6
  $ ./mvnw clean package
  ```

- Startup the Tomcat server and deploy web application

  ```
  $ ./mvnw cargo:run -P tomcat90
  ```

  > Note:
  >
  > We provide maven profiles per application server as follow:
  >
  > | Profile        | Description |
  > | -------------- | ----------- |
  > | tomcat90       | Running under the Tomcat 9.0 |
  > | tomcat85       | Running under the Tomcat 8.5 |
  > | tomee80        | Running under the TomEE 8.0(Java EE 8) |
  > | tomee71        | Running under the TomEE 7.1(Java EE 7) |
  > | wildfly26      | Running under the WildFly 26(Java EE 8) |
  > | wildfly13      | Running under the WildFly 13(Java EE 7) |
  > | liberty-ee8    | Running under the WebSphere Liberty(Java EE 8) |
  > | liberty-ee7    | Running under the WebSphere Liberty(Java EE 7) |
  > | jetty          | Running under the Jetty 9 |
  > | glassfish5     | Running under the GlassFish 5(Java EE 8) |
  > | glassfish4     | Running under the GlassFish 4(Java EE 7) |
  > | resin          | Running under the Resin 4 |

- Run application in browser http://localhost:8080/jpetstore/ 
- Press Ctrl-C to stop the server.

## Run on Docker
```
docker build . -t jpetstore
docker run -p 8080:8080 jpetstore
```
or with Docker Compose:
```
docker compose up -d
```

## Try integration tests

Perform integration tests for screen transition.

```
$ ./mvnw clean verify -P tomcat90
```
