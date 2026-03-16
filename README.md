# 🚀 Deploy Java Web App on Docker Container using Jenkins on AWS

## 📋 Project Overview
This project demonstrates a complete CI/CD pipeline that automatically builds 
and deploys a Java Web Application on a Docker Container hosted on AWS EC2, 
triggered by GitHub commits through Jenkins.

## 🏗️ Architecture
GitHub → Jenkins (EC2) → Maven Build → Docker Host (EC2) → Docker Container → Browser

## 🛠️ Technologies Used
- **AWS EC2** - Cloud infrastructure (2 instances)
- **Jenkins** - CI/CD automation server
- **Maven** - Java build tool
- **Docker** - Containerization
- **Git/GitHub** - Source code management
- **Apache Tomcat** - Web server (inside Docker)
- **Amazon Linux 2023** - OS

## ✅ Prerequisites
- AWS Account
- GitHub Account with source code
- Local machine with Git Bash / SSH client

---

## 📌 Phase 1: Setup Jenkins Server on AWS EC2

### Step 1: Launch EC2 Instance
- Instance type: t3.micro (free tier)
- Security Group: allow port 22 (SSH) and 8080 (Jenkins)
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
---> **successfully connected** to my EC2 instance via SSH ✅ :
  PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Step 2: Install Java 17 & Jenkins
```bash
sudo dnf install java-17-amazon-corretto-devel -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
Access Jenkins at: `http://<EC2-PUBLIC-IP>:8080`
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
---

## 📌 Phase 2: Install Git & Maven
```bash
# Git
sudo dnf install git -y
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
```bash
# Maven
cd /opt
sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.12/binaries/apache-maven-3.9.12-bin.tar.gz
sudo tar -xvzf apache-maven-3.9.12-bin.tar.gz
sudo mv apache-maven-3.9.12 maven
```
Set environment variables in `~/.bash_profile`:
```bash
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
export PATH
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

```bash
# Apply and verify:
source ~/.bash_profile
mvn -version
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

---

## 📌 Phase 3: Configure Jenkins Plugins & Tools

### Plugins installed:
Go to **Manage Jenkins → Plugins → Available plugins** and search for + install these one by one:
- GitHub Integration
- Maven Integration
- Publish Over SSH
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Global Tools configured:
Go to **Manage Jenkins → Tools** and set:
**Git:**
- Name: `Git`
- Path: `/usr/bin/git`
  PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

**JDK:**
- Click "Add JDK"
- Uncheck "Install automatically"
- Name: `Java17`
- JAVA_HOME: `/usr/lib/jvm/java-17-amazon-corretto.x86_64/`
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

**Maven:**
- Click "Add Maven"
- Uncheck "Install automatically"
- Name: `Maven`
- MAVEN_HOME: `/opt/maven`
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP


---

## 📌 Phase 4: Setup Docker Host EC2

### Step 1: Launch second EC2 instance
- Security Group: allow ports 22 and 8081-9000
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Step 2: Install Docker
```bash
sudo dnf install docker -y
sudo systemctl enable docker
sudo systemctl start docker
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Step 3: Create user and working directory
```bash
sudo useradd admin
sudo passwd admin
sudo usermod -aG docker admin
sudo mkdir /opt/docker
sudo chown -R admin:admin /opt/docker
```

### Step 4: Enable password SSH authentication
```bash
sudo vi /etc/ssh/sshd_config
```
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

```bash
# Set: PasswordAuthentication yes
sudo service sshd reload
```

### Step 5: Create Dockerfile
```bash
vi /opt/docker/Dockerfile
```
```dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY ./*.war /usr/local/tomcat/webapps
```

---

## 📌 Phase 5: Integrate Docker Host with Jenkins

### Install "Publish Over SSH" Plugin
In Jenkins → **Manage Jenkins → System → Publish Over SSH:**
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Configure Docker Host in Jenkins

PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

✅ Test Configuration → **Success**

---

## 📌 Phase 6: Create Jenkins Pipeline Job

### Create and configure of a New Jenkins Job
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

**Post-build Actions** (runs after everything):
- Used for actions that involve **external systems** like sending files to another server via SSH
- This is where **"Send build artifacts over SSH"** lives because it needs to transfer the `.war` file to our **Docker host EC2** which is a separate machine
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

Click **Save**.

### Trigger the Build
- ✅ Maven build succeeded
- ✅ **SSH: Transferred 1 file(s)** — WAR file copied to Docker host
- ✅ Docker build and container launched

PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Check WAR file is in the right place :
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
Checking if docker image was created and container is running : 
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
Now let's access to our app in the browser :
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

---

## 📌 Phase 7: Automate with GitHub Webhook

In GitHub repo → **Settings → Webhooks → Add webhook:**
- **Payload URL:** `http://<JENKINS-PUBLIC-IP>:8080/github-webhook/`
- **Content type:** `application/json`
- **Events:** Just the push event
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

Now every `git push` automatically triggers Jenkins! ✅

---

## 🎯 Result

We will make a change on README.md and commit :
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
The build triggered automatically : 
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
Once the build succeeds, the new Docker image and running container should appear as follows :
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
From the GitHub Hook Log we can see when and from which repository GitHub sent a notification to Jenkins, and whether Jenkins received it successfully or not.
In short — it's the proof that the automatic trigger between GitHub and Jenkins is working. ✅
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Full Pipeline Flow:
```
git push → GitHub Webhook → Jenkins triggered →
Maven builds WAR → SSH transfers to Docker Host →
Docker builds image → Container launched → App live!

## 👩‍💻 Author
**Afaf Oubella**  
DevOps Mini Project — March 2026
