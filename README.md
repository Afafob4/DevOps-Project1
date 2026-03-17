# 🚀 Deploy Java Web App on Docker Container using Jenkins on AWS

<img width="779" height="442" alt="image" src="https://github.com/user-attachments/assets/b41a9d28-6a6b-43ba-bd85-7ea3db18acce" />

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
<img width="906" height="383" alt="jenkins servers creation" src="https://github.com/user-attachments/assets/e8b3210e-9772-43b3-8deb-c8b708385d14" />
<img width="739" height="131" alt="Instance jenkins" src="https://github.com/user-attachments/assets/fee6506f-d016-45be-96eb-e9f7edb509e9" />

---> **successfully connected** to my EC2 instance via SSH ✅ :
  
  <img width="535" height="194" alt="Pasted image 20260301043553" src="https://github.com/user-attachments/assets/67fad565-8c20-4565-b748-afea60ddbe30" />
  
### Step 2: Install Java 17 & Jenkins
```bash
sudo dnf install java-17-amazon-corretto-devel -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
<img width="551" height="98" alt="Pasted image 20260302035256" src="https://github.com/user-attachments/assets/4f212f1e-90de-4551-937b-fe62f655c233" />

Access Jenkins at: `http://<EC2-PUBLIC-IP>:8080`

<img width="938" height="367" alt="Pasted image 20260302035555" src="https://github.com/user-attachments/assets/b2ca8888-1aa6-4814-b814-0931f908ecf5" />

---

## 📌 Phase 2: Install Git & Maven
```bash
# Git
sudo dnf install git -y
```
<img width="1612" height="454" alt="Pasted image 20260302035712" src="https://github.com/user-attachments/assets/45f18378-ef52-4b68-bdb2-8fb113689617" />

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
<img width="482" height="162" alt="Pasted image 20260302055510" src="https://github.com/user-attachments/assets/0f5e9dc6-9d4d-4507-bc73-253971642121" />

```bash
# Apply and verify:
source ~/.bash_profile
mvn -version
```
<img width="614" height="83" alt="Pasted image 20260302042411" src="https://github.com/user-attachments/assets/017dbdc2-041d-4f2e-a104-ad9d96df5861" />

---

## 📌 Phase 3: Configure Jenkins Plugins & Tools

### Plugins installed:
Go to **Manage Jenkins → Plugins → Available plugins** and search for + install these one by one:
- GitHub Integration
- Maven Integration
- Publish Over SSH

<img width="920" height="449" alt="Pasted image 20260302043000" src="https://github.com/user-attachments/assets/07e7a49f-667b-496c-8ec3-d47c8aa18918" />

### Global Tools configured:
Go to **Manage Jenkins → Tools** and set:
**Git:**
- Name: `Git`
- Path: `/usr/bin/git`
- 
<img width="1605" height="567" alt="Pasted image 20260302060738" src="https://github.com/user-attachments/assets/175a775d-58d7-41bd-ac0b-1c030361d25f" />

**JDK:**
- Click "Add JDK"
- Uncheck "Install automatically"
- Name: `Java17`
- JAVA_HOME: `/usr/lib/jvm/java-17-amazon-corretto.x86_64/`

<img width="786" height="344" alt="Pasted image 20260302060704" src="https://github.com/user-attachments/assets/23dc83ec-0f6a-41fd-a946-57e4772772c4" />

**Maven:**
- Click "Add Maven"
- Uncheck "Install automatically"
- Name: `Maven`
- MAVEN_HOME: `/opt/maven`

<img width="796" height="331" alt="Pasted image 20260302060811" src="https://github.com/user-attachments/assets/230ca428-f239-4bc6-b33c-50bcc3babc9e" />

---

## 📌 Phase 4: Setup Docker Host EC2

### Step 1: Launch second EC2 instance
- Security Group: allow ports 22 and 8081-9000

<img width="758" height="183" alt="Pasted image 20260311032632" src="https://github.com/user-attachments/assets/31fad647-d068-4c09-9063-bfd6ff087a1a" />

### Step 2: Install Docker
```bash
sudo dnf install docker -y
sudo systemctl enable docker
sudo systemctl start docker
```

<img width="602" height="210" alt="Pasted image 20260311032805" src="https://github.com/user-attachments/assets/92f2abc5-eb37-434d-89c2-342e07ffb885" />
<img width="649" height="67" alt="Pasted image 20260311033147" src="https://github.com/user-attachments/assets/e9724e16-63a3-4e9d-ad66-4c98631caf41" />

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

<img width="416" height="139" alt="Pasted image 20260311034458" src="https://github.com/user-attachments/assets/b09b0ec5-c2d0-444c-a7bf-b3c269ba873d" />

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

<img width="680" height="213" alt="Pasted image 20260311040813" src="https://github.com/user-attachments/assets/ee36d70a-b68e-4523-a374-e57352e22c91" />

✅ Test Configuration → **Success**

---

## 📌 Phase 6: Create Jenkins Pipeline Job

### Create and configure of a New Jenkins Job

<img width="941" height="425" alt="Pasted image 20260312032027" src="https://github.com/user-attachments/assets/efe3a003-4925-4e4c-a535-67f332119b32" />
<img width="674" height="355" alt="Pasted image 20260312032052" src="https://github.com/user-attachments/assets/b50c6f6b-6719-4ff7-a884-01800a5fc626" />
<img width="926" height="239" alt="Pasted image 20260312032121" src="https://github.com/user-attachments/assets/24541b3e-18ca-4fd5-b4cc-c7a4b5a4767b" />
<img width="905" height="324" alt="Pasted image 20260314021831" src="https://github.com/user-attachments/assets/21728408-f21f-4658-9724-5009e67e0dd4" />

**Post-build Actions** (runs after everything):
- Used for actions that involve **external systems** like sending files to another server via SSH
- This is where **"Send build artifacts over SSH"** lives because it needs to transfer the `.war` file to our **Docker host EC2** which is a separate machine

<img width="947" height="316" alt="Pasted image 20260312033657" src="https://github.com/user-attachments/assets/cdc84c41-114d-431b-8b87-1a907f4608a3" />
<img width="623" height="217" alt="Pasted image 20260312033759" src="https://github.com/user-attachments/assets/fa5fa15c-4454-4a3b-884c-c5d15490f507" />
<img width="938" height="455" alt="Pasted image 20260314053046" src="https://github.com/user-attachments/assets/c8d5ea7c-725d-4b36-85a9-cd4e64d0f72c" />

Click **Save**.

### Trigger the Build
- ✅ Maven build succeeded
- ✅ **SSH: Transferred 1 file(s)** — WAR file copied to Docker host
- ✅ Docker build and container launched

PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP

### Check WAR file is in the right place :

<img width="335" height="45" alt="Pasted image 20260314053512" src="https://github.com/user-attachments/assets/73daf151-7b7a-4fb1-b487-f26399b38255" />

Checking if docker image was created and container is running : 

<img width="883" height="105" alt="Pasted image 20260314054017" src="https://github.com/user-attachments/assets/e779a4a0-ee8c-4785-83ca-7a88c28cae73" />

Now let's access to our app in the browser :

<img width="851" height="367" alt="Pasted image 20260314054255" src="https://github.com/user-attachments/assets/63f0f6db-5363-4eaf-bde8-9f8f576242f1" />

---

## 📌 Phase 7: Automate with GitHub Webhook

In GitHub repo → **Settings → Webhooks → Add webhook:**
- **Payload URL:** `http://<JENKINS-PUBLIC-IP>:8080/github-webhook/`
- **Content type:** `application/json`
- **Events:** Just the push event

<img width="932" height="452" alt="Pasted image 20260314064408" src="https://github.com/user-attachments/assets/65c3fdfd-81d3-4458-a0ce-6be7ef54744c" />

Now every `git push` automatically triggers Jenkins! ✅

---

## 🎯 Result

We will make a change on README.md and commit :

<img width="940" height="278" alt="Pasted image 20260314064615" src="https://github.com/user-attachments/assets/d2b361b1-d7f7-47ff-9c89-e4a42d7bd57d" />

The build triggered automatically : 

<img width="725" height="386" alt="Pasted image 20260314064913" src="https://github.com/user-attachments/assets/d5dedffb-b8f5-4900-b6a8-f1e23c88be3f" />
<img width="765" height="292" alt="Pasted image 20260314071432" src="https://github.com/user-attachments/assets/2ceb4516-8b25-46c2-b7e1-75e1586db4f1" />

Once the build succeeds, the new Docker image and running container should appear as follows :

<img width="886" height="122" alt="Pasted image 20260314072353" src="https://github.com/user-attachments/assets/20cd424e-c6ed-42c3-8fbc-173bf9a4ffeb" />
<img width="886" height="122" alt="Pasted image 20260314072405" src="https://github.com/user-attachments/assets/db1e3b59-113f-487f-a47a-9f239b351824" />

From the GitHub Hook Log we can see when and from which repository GitHub sent a notification to Jenkins, and whether Jenkins received it successfully or not.
In short — it's the proof that the automatic trigger between GitHub and Jenkins is working. ✅

<img width="766" height="287" alt="Pasted image 20260314072532" src="https://github.com/user-attachments/assets/b4cf302b-34ef-4025-87f1-f551d26dfe43" />

# 🎉 CONGRATULATIONS! The project is COMPLETE!

Our Java web application is successfully deployed and running on a Docker container on AWS EC2, fully automated through Jenkins!

Here's what we accomplished:

**The full pipeline is working:**

- ✅ Code hosted on **GitHub**
- ✅ **Jenkins** automatically pulls code, builds with **Maven**
- ✅ WAR file transferred to **Docker host** via SSH
- ✅ **Docker image** built automatically
- ✅ **Docker container** launched and serving the app
- ✅ App accessible at `http://<DOCKER-HOST-PUBLIC-IP>:8087/webapp/`
**The automation works end-to-end** — any time we push code to GitHub, Jenkins will automatically rebuild and redeploy the app without any manual intervention!

### Full Pipeline Flow:
```
git push → GitHub Webhook → Jenkins triggered →
Maven builds WAR → SSH transfers to Docker Host →
Docker builds image → Container launched → App live!
```
