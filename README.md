Absolutely! Here's the **entire process combined into a single README.md** for easy copy-paste:

````markdown
# Jenkins Setup on Ubuntu 24.04 (t2.large) AWS Instance

This guide shows how to launch an Ubuntu 24.04 instance on AWS, connect to it, update packages, install AWS CLI, install Jenkins, and access Jenkins on port 8080.

---

## 1. Launch Instance

- Ubuntu 24.04
- Instance type: t2.large
- Storage: Minimum 30 GB

---

## 2. Connect to Instance

```bash
ssh -i your-key.pem ubuntu@your-instance-public-ip
````

---

## 3. Update Packages

```bash
sudo su
sudo apt update -y
```

---

## 4. Install AWS CLI

```bash
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

---

## 5. Install Jenkins

```bash
sudo apt update -y

# Add Adoptium repo and install Java 17
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list

sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version

# Add Jenkins repo and install Jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y
sudo apt-get install jenkins -y

# Start Jenkins service
sudo systemctl start jenkins
sudo systemctl status jenkins

# Verify Jenkins version
jenkins --version
```

---

## 6. Open Port 8080

* In AWS Console, go to EC2 > Security Groups
* Edit inbound rules and add:

  * Type: Custom TCP
  * Port: 8080
  * Source: Your IP or 0.0.0.0/0 (for public access)

---

## 7. Access Jenkins UI

Open in browser:

```
http://your-instance-public-ip:8080
```

Sure! Here's the full **combined README.md** with steps 6 to 13 included for easy copy-paste:


## 8. Install Docker on Ubuntu

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
newgrp docker
sudo systemctl status docker

docker --version
````

---

## 9. Install Trivy (Vulnerability and Misconfiguration Scanner)

```bash
sudo apt-get install -y wget apt-transport-https gnupg

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install -y trivy

trivy --version
```

---

## 10. Install Docker Scout (Image Insights & Supply Chain Security)

1. Log in to DockerHub from your terminal:

```bash
docker login -u <DockerHubUserName>
# Enter your DockerHub password when prompted
```

2. Install Docker Scout CLI:

```bash
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
sudo chmod 777 /var/run/docker.sock
```

---

## 11. Install SonarQube using Docker

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
docker ps
```

Access SonarQube UI at `http://your-instance-public-ip:9000`

---

## 12. Install Jenkins Plugins

Install the following plugins from Jenkins Plugin Manager:

* Pipeline: Stage View (2.38)
* Eclipse Temurin installer (1.7)
* SonarQube Scanner (2.18)
* Docker (1274.vc0203fdf2e74)
* Docker Commons (451.vd12c371eeeb\_3)
* Docker Pipeline (611.v16e84da\_6d3ff)
* Docker API (3.5.0-108.v211cdd21c383)
* docker-build-step (2.12)
* Email Extension Template (233.v1eb\_88fc160b\_5)
* OWASP Dependency-Check (5.6.1)
* NodeJS (1.6.4)
* Prometheus metrics (819.v50953a\_c560dd)

---

## 13. SonarQube Configuration in Jenkins

### 13.1 Generate SonarQube Token

* Go to SonarQube UI → Administration → Security → Users → Tokens
* Create token with:

  * Name: `jenkins`
  * Expiration: 30 days
* Copy the generated token.

### 13.2 Tools Configuration in Jenkins

* Jenkins → Manage Jenkins → System Configuration → Tools
* Add **JDK**:

  * Name: `jdk17`
  * Check **Install automatically**
  * Add Installer → Install from Adoptium.net → Version: `jdk-17.0.8.1+1`
* Add **SonarQube Scanner**:

  * Name: `sonar-scanner`
  * Version: Latest (e.g., 6.2.0.4584)
* Add **NodeJS**:

  * Name: `node23`
  * Version: `16.20.0`
* Add **Dependency-check**:

  * Name: `DP-Check`
  * Check **Install automatically**
  * Add Installer → Install from GitHub.com → Latest version (e.g., 10.0.4)
* Add **Docker**:

  * Name: `docker`
  * Check **Install automatically**
  * Add Installer → Download from docker.com → Latest version
* Apply and Save

### 13.3 Configure SonarQube Token Credentials in Jenkins

* Jenkins → Manage Jenkins → Security → Credentials
* Add new **Secret Text** credential:

  * Name & Description: `Sonar-token`
  * Secret: *Paste SonarQube token*

### 13.4 Configure DockerHub Credentials in Jenkins

* Jenkins → Manage Jenkins → Security → Credentials
* Add new **Username and Password** credential:

  * Username: DockerHub username
  * Password: DockerHub password or token
  * Name & Description: `docker`

### 13.5 Configure Email Notification Credentials

* Generate email passkey (e.g., via [https://myaccount.google.com/signinoptions/passkeys](https://myaccount.google.com/signinoptions/passkeys))
* Jenkins → Manage Jenkins → Security → Credentials
* Add new **Username and Password** credential:

  * Username: your email
  * Password: generated passkey
  * Name & Description: `email-creds`

---

## 14. Jenkins System Configuration

### Configure SonarQube Server

* Jenkins → Manage Jenkins → System → SonarQube Servers
* Add:

  * Name: `sonar-server`
  * Server URL: `http://your-sonarqube-ip:9000` (no trailing slash)
  * Authentication Token: select `Sonar-token`

### Configure Extended Email Notification

* Jenkins → Manage Jenkins → System → Extended E-mail Notification
* SMTP Server: `smtp.gmail.com`
* SMTP Port: `465`
* Advanced settings:

  * Use Credentials: select `email-creds`
  * Use SSL: enabled
  * Use OAuth 2.0: enabled

### Configure Email Notification

* Jenkins → Manage Jenkins → System → E-mail Notification
* SMTP Server: `smtp.gmail.com`
* Advanced:

  * User Name: your email
  * Password: email passkey (from `email-creds`)
  * Use SSL: enabled
  * SMTP Port: `465`
  * Reply-To Address: your email
  * Charset: `UTF-8`
* Test email configuration

### Configure Default Build Triggers

* Jenkins → Manage Jenkins → System → Default triggers
* Select: Failure-Any, Always, Success
* Apply and Save

---

## 15. Create Webhook in SonarQube

* SonarQube → Configurations → Webhooks → Create

  * Name: `Jenkins`
  * URL: `http://<jenkins-ip>:<jenkins-port>/sonarqube-webhook/`
* Save





