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




