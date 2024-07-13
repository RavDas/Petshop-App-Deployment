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


### 1. Create an Ubuntu(22.04) T2 Large Instance - 30GB Storage (To install Jenkins with plugins, SonarQube, Trivy)

Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable ssh (22), HTTP(80) and HTTPS(443) traffic by opening ports in the instance settings in the Security Group.

![image](https://github.com/user-attachments/assets/a61ddcd3-8913-42a2-9bf9-97c3948da5fa)

![image](https://github.com/user-attachments/assets/85ac7c7d-67c1-4c7a-8d2c-645adfc22cde)

![Uploading image.png…]()


**OR**

ssh into the EC2 instance using GUI of the MobaXtreme termminal like below.

![image](https://github.com/user-attachments/assets/2ae5a416-c1fc-4381-a04c-84cecfee8095)


Step 2 — Install Jenkins, Docker and Trivy

2A — To Install Jenkins

Connect to your console, and enter these commands to Install Jenkins

```
vi jenkins.sh
```

```
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
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
sudo vi jenkins   #chnage port HTTP_PORT=8090 and save and exit
cd /lib/systemd/system
sudo vi jenkins.service  #change Environments="Jenkins_port=8090" save and exit
sudo systemctl daemon-reload
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

Now, grab your Public IP Address

```
<EC2 Public IP Address:8090>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```







































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
