![Untitled design (2)](https://github.com/user-attachments/assets/ebc66a05-fdb5-41e4-8aa6-79b4d889ff99)


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

Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. 

Enable below ports in the security group of the Instance,

* SSH: Port 22 (TCP) - Used for secure shell access.
* HTTP: Port 80 (TCP) - Used for unencrypted web traffic.
* HTTPS: Port 443 (TCP) - Used for encrypted web traffic.
* Jenkins: Port 8080 (TCP) - Default HTTP port for Jenkins web interface. (3000-10000)
* Kubernetes:
       - API Server: Port 6443 (TCP) - Kubernetes API server. (3000-10000)
       - NodePort Services: Ports typically in the range 30000-32767 (TCP/UDP) for NodePort services.

* Docker: Port 2376 (TCP) - Docker daemon over TLS.
* SonarQube: 9000 (3000-10000)
* Prometheus: Port 9090 (TCP) - Default port for Prometheus metrics server. (3000-10000)
* Grafana: Port 3000 (TCP) - Default port for Grafana web interface. (3000-10000)
* Node Exporter: Port 9100 (TCP) - Default port for Prometheus Node Exporter. (3000-10000)
* SMTP (Plain Text): 25
* SMTP (Secure/Encrypted - SMTPS): 465
  
I have open ports from 3000-10000 port range for ease as most of the required traffic are within that range.

![image](https://github.com/user-attachments/assets/2dbc42ae-3b59-4227-9e45-53ac655ae01a)


![1 55](https://github.com/user-attachments/assets/0d7690d3-bf7f-4f1f-89cf-5b3f3fb29ce6)

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
git clone https://github.com/RavDas/Petstore-App-Deployment.git
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
git remote add new-origin https://github.com/RavDas/Petstore-App-Deployment.git
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

Since Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080(default port) and 9000 for Jenkins and SonarQube to work. (We already opened the relavant ports)

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

#### 3 - Install Trivy

```
vi trivy.sh
```
```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

Execute the ```trivy.sh``` file

```
sudo chmod +x trivy.sh 
```

Run the ```trivy.sh``` file

```
./trivy.sh
```

Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins


### Step 3 — Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check


#### 3A — Install Plugin

Goto Manage Jenkins -> Plugins -> Available Plugins

Install below plugins

* Eclipse Temurin Installer (Install without restart)

* SonarQube Scanner (Install without restart)

![image](https://github.com/user-attachments/assets/1d086abb-b6e9-4445-8433-ff414aaf7a19)


#### 3B — Configure Java and Maven in Global Tool Configuration

Goto Manage Jenkins -> Tools -> Install JDK(17) and Maven3(3.6.0) -> Click on Apply and Save

![image](https://github.com/user-attachments/assets/617e58c1-ddba-403d-811b-79f770b8980f)


![image](https://github.com/user-attachments/assets/9e3728d1-cb1c-4808-807d-32a59d642058)


#### 3C — Create a Job

Label it as "petstore", click on Pipeline and OK.

![image](https://github.com/user-attachments/assets/fc55a5fb-9268-4c0b-b5a1-a094273dc243)

Then you will enter to "Configurations" page of the created pipepline. In "General" section under "Configuration";

Tick on "Discard old builds" and set "Max # of build to keep" to 3.

Then enter below Pipeline Script in the "Pipeline" section ,

```
pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git 'https://github.com/RavDas/Petstore-App-Deployment.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
   }
}
```

The "Apply" and "Save"

The stage view would look like this,

![image](https://github.com/user-attachments/assets/8f2b3c91-bc0a-43cf-a923-de9add3855a9)



### Step 4 — Configure Sonar Server in Manage Jenkins

Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000.

Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

![image](https://github.com/user-attachments/assets/860e5557-b946-4fa3-bdf8-832031881b78)

Click on Update Token

![image](https://github.com/user-attachments/assets/98e9b6e0-2453-4784-986f-c83dfd3f6b28)

Create a token with a Name and Generate

![image](https://github.com/user-attachments/assets/ada24cc0-7071-458d-a56f-3572719c6ffc)

Copy Token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Click on global → Add Credentials

It should look like this,

![image](https://github.com/user-attachments/assets/91caf9dc-3b14-423e-ad0b-9f77a1f59cc9)

You will this page once you click on create

![image](https://github.com/user-attachments/assets/11444580-b0b8-4582-8c7b-4d3867e9a9a5)


Now, go to Dashboard → Manage Jenkins → System and Add like the below image.

![image](https://github.com/user-attachments/assets/5e8dc85e-8383-4879-9cbb-4c5e870e7679)

Click on Apply and Save.


**System Configurion** option is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a SonarQube Scanner in the Manage → Tools.

![image](https://github.com/user-attachments/assets/8c294deb-9100-4ed7-a652-7606c45ae823)

Click on Apply and Save.


In Sonarqube Dashboard, quality gate should be added.

![1 2](https://github.com/user-attachments/assets/6065c31e-f1f1-48d2-8a82-7b34f76467dc)

Administration –> Configuration –> Webhooks

![image](https://github.com/user-attachments/assets/cd62f836-1281-4e6c-8903-bcde690f7138)

Click on Create

![image](https://github.com/user-attachments/assets/ed9c7209-d72d-475a-9dca-55c5ac8a2cf6)

Add details

![image](https://github.com/user-attachments/assets/a0e53c3c-4d82-4a25-8992-cda0894198ab)


```
#in url section of quality gate
<http://jenkins-public-ip:8090>/sonarqube-webhook/
```


Let’s go to our Pipeline and add Sonarqube Stage in our Pipeline Script.


```
#under tools section add this environment
environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

# in stages add this
        stage("Sonarqube Analysis "){
            steps{
              withSonarQubeEnv('sonar-server') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petstore \
                  -Dsonar.java.binaries=. \
                  -Dsonar.projectKey=Petstore '''ge 
              }
            }
        }

        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
           }
        }
```

In above we add "quality gate" to stop the pipeline building further if "Sonarqube Analysis" fails.

Click on Build now, you will see the stage view like this,

![image](https://github.com/user-attachments/assets/a68bb2c1-f89e-49c4-87fe-140bbfb737d8)


To see the report, you can go to Sonarqube Server and go to Projects.

![image](https://github.com/user-attachments/assets/46d8c81a-2571-444d-b4ed-73010ad33fba)


You can see the report has been generated and the status shows as passed. You can see that there are 6.7k lines. To see a detailed report, you can go to issues.

### Step 5 — Install OWASP Dependency Check Plugins

Goto Dashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

![image](https://github.com/user-attachments/assets/f6bedcee-adba-4dc7-b265-aa1cd557b898)

First, we configured the Plugin and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools 

![image](https://github.com/user-attachments/assets/6e5696ad-484f-4878-b180-ee4402f56487)

Click on Apply and Save here.

Now go configure → Pipeline and add this stage to your pipeline and build.


```
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```


The stage view would look like this,

![image](https://github.com/user-attachments/assets/6d782e9f-6cc3-442a-8958-093f56bebed9)

You will see that in status, a graph will also be generated and Vulnerabilities.

![image](https://github.com/user-attachments/assets/c82ff299-7c32-450e-80b9-7dbe33c5251f)



### Step 6 — Docker plugin and credential Setup

We need to install the Docker tool in our system, 

Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

* Docker

* Docker Commons

* Docker Pipeline

* Docker API

* docker-build-step

and click on install without restart

![image](https://github.com/user-attachments/assets/99c09464-7b2c-4e18-9864-41b7f3a3d65b)

Now, goto Dashboard → Manage Jenkins → Tools 

![image](https://github.com/user-attachments/assets/b5e07545-d5d7-43d4-8fc7-0fa1edad8cb1)


Add DockerHub Username and Password under Global Credentials

![1](https://github.com/user-attachments/assets/b67292d0-2bfd-4424-851c-2ff6f645eb4e)

Instead of adding the password, you can create a personal Access token from the Docker Hub which can also be used for ansible-playbook.

![2](https://github.com/user-attachments/assets/a9f5e70c-5e85-47f7-b5a3-14f6c9ab9331)

Generate and copy it and save for later(It will works as a password credential of Docker Hub).

![access](https://github.com/user-attachments/assets/d0a87bc1-ac33-408d-9102-7cf6b886b30d)


Now we have two credential in the "Global Credentials" section.

![11](https://github.com/user-attachments/assets/6272207d-3f51-4fd9-b6ce-1fcf43f50036)



### STEP 7 -Adding Ansible Repository in Ubuntu

Setting up Master (Controlled node)-Slave (Managed node) using Ansible

Master-Slave architecture is a common setup in configuration management where the master node controls and manages multiple slave nodes. In Ansible, this can be efficiently achieved with minimal configuration:

Here the public IP of the Jenkins and the Ansible are same because we have Jenkins and Ansible on the same EC2 instance. So we have not created a master-slave for Ansible. We used a single Ansible, and that too is in our Jenkins machine.

1. Install Ansible: Ensure Ansible is installed on the master node (in our case Jenkins/Ansible machine).

Run the below commands on Jenkins server EC2 instance (Ubuntu 22.04 LTS) to add Ansible repository.

Update your system packages:

```
sudo apt-get update
```

First Install Required packages to install Ansible.

```
sudo apt install software-properties-common
```

![image](https://github.com/user-attachments/assets/97f9931d-edfa-4604-be38-2b656b2046be)

Add the ansible repository via PPA

```
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

![image](https://github.com/user-attachments/assets/7694ee08-271c-4743-9f7d-a9c5cc13518a)


Install Python3 on the Ansible master

```
sudo apt install python3
```

![image](https://github.com/user-attachments/assets/126c849c-bfb6-41f8-bcd2-42dc9a7209cc)

Install Ansible on our Jenkins EC2 instance.

```
sudo apt install ansible -y
``` 

![image](https://github.com/user-attachments/assets/06194998-537f-4d8c-8844-63d0b1b34111)

```
sudo apt install ansible-core -y
```

![image](https://github.com/user-attachments/assets/0836e1cf-7cb5-4437-bf2b-7f77035474ec)

To check version :

```
ansible --version
```

2. Create an Inventory File: Create an inventory file listing all slave nodes in '/etc/ansible/hosts' of master node (in our case Jenkins/Ansible machine).

To add inventory, you can create a new directory or add in the default Ansible hosts file. 

```
cd /etc/ansible
sudo vi hosts
```

Now go to the host file inside the Ansible server and paste the public IP of the Jenkins. 

![image](https://github.com/user-attachments/assets/e8a8e9b7-6fe8-4106-9377-6a3553e19f73)


You can create a group and paste IP address below:

```
[local] #any name you want
IP of Jenkins
```

(Optional - How to make changes in a ```vi``` fiel and save it.)
To enter Insert mode, press i .
To exit Insert mode, press Esc key.
Type ':x' and press Enter key to save and exit the```vi``` file.


![image](https://github.com/user-attachments/assets/2bc981ba-d787-4052-a164-94d4243529bd)

Save and exit from the file.

3. SSH Configuration: Set up SSH keys for passwordless authentication from the master to all slave nodes (in our case both master and slave are Jenkins/Ansible machine).

This setup allows the master node to send commands and configurations to slave nodes, maintaining a unified and consistent environment.   

Let’s install The Ansible plugin to integrate with Jenkins.

![image](https://github.com/user-attachments/assets/7e77b3ad-07b6-4a10-ba0c-f0bb9ee0f3bc)

Now add Credentials to invoke Ansible with Jenkins.

![image](https://github.com/user-attachments/assets/2176d291-96b1-408a-b876-7d0f54be8d9d)


In the "Private Key" section of "Credentials", Select "Enter directly" and add your .pem file ( of the Jenkins EC2 Instance ) content for the "Key".

![image](https://github.com/user-attachments/assets/65b6d44e-25e4-4c25-bef0-f563c12016ac)

and finally, click on Create.

So now there are 3 "Global credentials".

![1 2](https://github.com/user-attachments/assets/84ea01fc-0055-4afe-9d91-0f0085f1ef4c)

Give this command in your Jenkins machine to find the path of your Ansible which is used in the "Tools" section of Jenkins.

```
which ansible
```

![image](https://github.com/user-attachments/assets/ef68fbef-4f60-431f-9258-0229cf0cd77c)


Copy that path and add it to the "Tools" section of Jenkins at ansible installations.

![image](https://github.com/user-attachments/assets/ac4bde5a-311c-48e3-a7d5-8af9378c8f8c)


Now write an Ansible playbook to create a docker image, tag it and push it to the docker hub, and finally, we will deploy it on a container using Ansible. This playbook is available in the Ansible/dockey.yaml of the Github repository.

```
- name: docker build and push
  hosts: local  # Replace with the hostname or IP address of your target server 
  become: yes  # Run tasks with sudo privileges
  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes
    - name: Build Docker Image
      command: docker build -t petstore .
      args:
        chdir: /var/lib/jenkins/workspace/petstore   #path of the Jenkins pipeline in workspace
    - name: tag image
      command: docker tag petstore:latest ravdas/petstore:latest #change <ravdas> to your Docker Hub username
    - name: Log in to Docker Hub
      community.docker.docker_login:
        registry_url: https://index.docker.io/v1/
        username: ravdas #change to your docker hub username
        password: <docker pat> #paste the generated access-key from the Docker Hub
    - name: Push image
      command: docker push ravdas/petstore:latest #change <ravdas> to your Docker Hub username
    - name: Run container
      command: docker run -d --name petstore -p 8081:8080 ravdas/petstore:latest  #change <ravdas> to your Docker Hub username
```

![1 3](https://github.com/user-attachments/assets/ea3bfd75-f62e-4b9f-9086-95a4fc3dd33d)

Add this stage to the pipeline to build and push it to the docker hub, and run the container.


```
    stage('Install Docker') {
                steps {
                    dir('Ansible'){
                       script {
                             ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                        }
                    }
                }
    }

```

Output of pipeline

![image](https://github.com/user-attachments/assets/aca8bacc-ba5a-4625-b56b-1b634ebb58ef)


Now copy the IP address of Jenkins and paste it into the browser

```
<jenkins-ip:8081>/jpetstore
```

![1 4](https://github.com/user-attachments/assets/7821bfc1-da13-46ad-87d0-a905598c7142)


### Step 8 — Kuberenetes Setup

Install Kubectl on Jenkins machine (Firstly created EC2 instance) also.

Create a shell script

```
sudo vi kube.sh
```

Copy below code part into the ```kube.sh``` file.

```
sudo apt update
sudo apt install curl
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

Make the .sh file executable

```
sudo chmod +x kube.sh
```

Run the executable file

```
./kube.sh
```

Execute two Ubuntu 20.04 instances one for k8s master and the other one for worker. (t2.medium and 8GB storage)

![1 45](https://github.com/user-attachments/assets/477e1a8b-d150-440e-b38f-078915e44bda)

Connect your both master and worker machines to Putty or Mobaxtreme using ssh like earlier using the keypair(.pem) file.

![Screenshot from 2024-07-14 18-41-46](https://github.com/user-attachments/assets/c1972f1e-4c1e-41a7-8116-bdafd5a46153)


8.1. On Master Node

```
sudo su
hostname master
bash
clear
```

8.2. On Worker Node

```
sudo su
hostname worker
bash
clear
```

8.3. On both Master & Worker Nodes

```
sudo apt-get update 
sudo apt-get install -y docker.io
sudo usermod –aG docker Ubuntu
newgrp docker
sudo chmod 666 /var/run/docker.sock #The Docker daemon communicates with clients through a Unix socket (/var/run/docker.sock)
```

Add a GPG key provided by Google Cloud, which is used to authenticate packages from the official Kubernetes repository hosted by Google.

```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Download official Kubernetes repository hosted on Google Cloud, which provides stable and tested Kubernetes packages.

```
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
```

```
sudo apt-get install -y kubelet kubeadm kubectl
```

8.4. On Master Node

Initialize Kubernetes Cluster

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
# in case your in root exit from it and run below commands (Running Kubernetes components as root can lead to permission issues)
```

Set Up Kubectl Configuration

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![1 9](https://github.com/user-attachments/assets/aeaeaffa-9bb8-4304-b3b0-6d2c7c68187c)

Apply a networking solution like Flannel to enable pod-to-pod communication.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

8.5. On Worker Node

```
sudo kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>
```

![Untitled design (1)](https://github.com/user-attachments/assets/28c6a9a2-c24f-4a0d-909e-ab8fa1181d1b)

![1 2](https://github.com/user-attachments/assets/c21bd636-4eb3-4d3a-b236-8fdb5cd8e863)

Check whether the nodes (instances of master and worker nodes) are created and running.

![image](https://github.com/user-attachments/assets/f4dd68dc-f1d0-423a-92cf-d9fdcda39a52)


Copy the config file to Jenkins master or the local file manager and save it to make the connection between Jenkins and Kubernetes.

```
cd .kube/

cat config
```
![image](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/96ffde47-0a5f-4097-b0ff-2ff2176cd11c)


Note: Create a secret-file.txt in your file explorer save the config file in it and use this at the Kubernetes "Credential" section.

Install Kubernetes Plugins in Jenkins, once it’s installed successfully.

![image](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/900bc68b-4114-46d0-ad39-797a0aa34bf5)


Go to manage Jenkins –> Manage Credentials –> Click on Jenkins global –> Add Credentials

![image](https://github.com/RavDas/Netflix-Clone-Deployment/assets/86109995/67f3859b-c28f-4d2e-b544-8baac3b1b634)



### STEP 9 — Master-slave Setup for Ansible and Kubernetes

To communicate with the Kubernetes clients we have to generate an SSH key on the Ansible master node(in our case it's Jenkins/Ansible) and exchange it with K8s Master System.

```
ssh-keygen
```

![image](https://github.com/user-attachments/assets/5f847318-6794-4eba-b36f-ffde325ab8ba)

Change the directory to .ssh and copy the public key (id_rsa.pub)

```
cd .ssh
cat id_rsa.pub  #copy this public key
```

![image](https://github.com/user-attachments/assets/e936f119-a457-4b32-b8b9-d2828e37cf16)

Once you copy the public key from the Ansible master, move to the Kubernetes master and change the directory to .ssh and paste the copied public key under authorized_keys.

```
cd .ssh #on k8s master 
sudo vi authorized_keys
```

![image](https://github.com/user-attachments/assets/e0afe445-b9b2-495c-948a-1e960ae3cb6e)

Note: Now, insert or paste the copied public key into the new line. Make sure you don’t delete any existing keys from the authorized_keys file then save and exit.

By adding a public key from the master to the Kubernetes machine, we have now configured keyless access. To verify you can try to access the Kubernetes master and use the command as mentioned as below in the Ansible server.

```
ssh ubuntu@<public-ip-kubernetes-master>
```

![image](https://github.com/user-attachments/assets/09e0556e-3034-45bb-8034-6a3a55f418a4)

Verifying the above SSH connection.

Now go to the host file inside the Ansible server and paste the public IP of the Kubernetes master.

![image](https://github.com/user-attachments/assets/55bafbed-258b-4bed-8c22-7e0e00b31cea)

You can create a group and paste ip address below:

```
[k8s]#any name you want
<public ip of kubernetes-master>
```

Test Ansible Server - Kubernetes Master node connection.(Here Ansible machine works as a master(controlled node) while Kubernetes master machine works as a slave(managed node) from the aspect of Ansible)

Use the below command to check Ansible Server - Kubernetes Master node connection.

```
ansible -m ping k8s
ansible -m ping all#use this one
```

If all configuration is correct then you would get below output.

![image](https://github.com/user-attachments/assets/c7e5cb6d-f740-4510-9004-0b112cffafc0)


Let’s create a simple ansible playbook for Kubernetes deployment.


---
- name: Deploy Kubernetes Application
  hosts: k8s  # Replace with your target Kubernetes master host or group
  gather_facts: yes  # Gather facts about the target host

  tasks:
    - name: Copy deployment.yaml to Kubernetes master
      copy:
        src: /var/lib/jenkins/workspace/petstore/deployment.yaml  # Assuming Jenkins workspace variable
        dest: /home/ubuntu/
      become: yes  # Use sudo for copying if required
      become_user: root  # Use a privileged user for copying if required

    - name: Apply Deployment
      command: kubectl apply -f /home/ubuntu/deployment.yaml
```

The ```src``` file path, that is deployment.yaml file is available in ```/var/lib/jenkins/workspace/petstore/deployment.yaml``` of the Ansible server should be copied to Kubernetes master node for deployment.

![1 3](https://github.com/user-attachments/assets/abeee788-b460-406d-b708-47dda0a5cfbc)


Now add the below stage to your pipeline.

```
       stage('k8s using ansible'){
              steps{
                     dir('Ansible') {
                           script{
                               ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                           }
                       }
              }
       }
```

Output,

![1 4](https://github.com/user-attachments/assets/77750566-c3df-411e-b7cc-08673963cb02)

In the Kubernetes cluster, give this command

```
kubectl get all
kubectl get svc
```

![image](https://github.com/user-attachments/assets/b9c42175-f651-4526-9fff-107ea2b8de86)


Type below to view the deployed application.

```
<kubernetesslave-ip:serviceport(30699)>/jpetstore
```

![1 65](https://github.com/user-attachments/assets/d61a734b-4273-4a22-82e2-635f51e52628)


============================================================================================================



Complete Pipeline

```
pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git 'https://github.com/Aj7Ay/jpetstore-6.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petstore \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petstore '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
           }
        }
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage('Ansible docker Docker') {
            steps {
                dir('Ansible'){
                  script {
                        ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                    }
                }
            }
        }

       stage("Trivy image scan"){
            steps{
                sh "trivy image ravdas/petstore:latest > trivyimage.txt" 
            }
        }
        
        stage('k8s using ansible'){
            steps{
                dir('Ansible') {
                    script{
                        ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                    }
                }
            }
        }
   }
}
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
