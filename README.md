Absolutely! Here's the **entire process combined into a single README.md** for easy copy-paste:


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
After installing docker apply following command to use docker demon in jenkins

```
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
To test if docker demon is working in jenkins run follwoing command

```
sudo su - jenkins
docker ps
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


#### 🧾 16. Create Pipeline Job

1. Open Jenkins → Click on **New Item** → Select **Pipeline** → Name it `zomato-pipeline`.
2. In the **Pipeline** configuration:

   * Scroll to the **Pipeline Script** section.
   * Paste the modified pipeline script below:

     * Replace the following placeholders:

       * `<image-name>` → e.g., `zomato-app`
       * `<your-docker-name>` → Your **DockerHub username**, e.g., `nishant9820`
       * `'your-email-id@gmail.com'` → Your **configured email ID in Jenkins**

---

### ✅ Example Modified Pipeline Script

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage ("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git '<your-github-URL>'
            }
        }
       stage("Sonarqube Analysis") {
    steps {
        withSonarQubeEnv('sonar-server') {
            sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectName=zomato \
                -Dsonar.projectKey=zomato \
                -Dsonar.sources=src \
                -Dsonar.inclusions=**/*.js,**/*.jsx \
                -Dsonar.exclusions=src/components/CTA/CTA.jsx,**/*.test.js,**/__tests__/**,**/node_modules/** \
             '''
           }
         }
       }
        stage("Code Quality Gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
         stage("Clean & Install Dependencies") {
        environment {
            JAVA_OPTS = '-Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400'
        }
        steps {
        sh "npm cache clean --force"
        sh "rm -rf node_modules package-lock.json"
        timeout(time: 10, unit: 'MINUTES') {
            sh "npm install --legacy-peer-deps --no-audit --no-optional --no-progress"
           }
         }
       }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato-app ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag <image-name> <your-docker-name>/<image-name>:latest"
                        sh "docker push <your-docker-name>/<image-name>:latest"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker-scout quickview <your-docker-name>/<image-name>:latest'
                        sh 'docker-scout cves <your-docker-name>/<image-name>:latest'
                        sh 'docker-scout recommendations <your-docker-name>/<image-name>:latest'
                    }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name <image-name> -p 3000:3000 <your-docker-name>/<image-name>:latest'
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                        <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                        </div>
                    </body>
                    </html>
                """,
                to: 'your-email-id@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy.txt'
        }
    }
}
```

---

📝 **Notes**:

* Ensure Jenkins is configured with:

  * Docker installed and running.
  * SonarQube configured with the `sonar-server` name.
  * Email notifications properly set up via `Manage Jenkins → Configure System → Extended E-mail Notification`.
* Credentials required in Jenkins:

  * DockerHub credentials ID as `docker`.
  * SonarQube token as `Sonar-token`.

---

# 📊 17. Monitoring Setup with Prometheus on Ubuntu 24.04

This guide helps you set up a **Monitoring Server** using **Prometheus**, **Grafana**, and **Node Exporter** on an AWS EC2 instance (Ubuntu 24.04).

---

## 🚀 Step 1: Launch EC2 VM for Monitoring

- **Name**: `Monitoring Server`
- **OS**: Ubuntu 24.04
- **Instance Type**: `t2.large`
- **Security Group**: Use the one created in Step 1 of your project
- **Storage**: 30GB EBS

---

## 🔌 Step 2: Connect to Your Monitoring Server

```bash
ssh -i <your-key.pem> ubuntu@<your-server-ip>
````

---

## 🛠️ Step 3: Install Prometheus

### 3.1 Create Prometheus User

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
```

### 3.2 Download and Extract Prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
```

### 3.3 Move Binaries and Config Files

```bash
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

### 3.4 Set Ownership

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

---

## ⚙️ Step 4: Create Prometheus Systemd Service

```bash
sudo vi /etc/systemd/system/prometheus.service
```

### Paste the following into the file:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

#### 🔍 Explanation:

* `User` and `Group`: Run Prometheus as the `prometheus` user.
* `ExecStart`: Defines how Prometheus starts, including config and data paths.
* `--web.listen-address`: Listens on all interfaces at port `9090`.
* `--web.enable-lifecycle`: Allows API-based config reloads.

---

## ▶️ Step 5: Start Prometheus

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

---

## 🌐 Step 6: Access Prometheus Dashboard

Open your browser and go to:

```
http://<your-server-ip>:9090
```

> 🔧 If `https://` doesn't work, remove the `s` and use `http://`.

---

## ✅ Step 7: Verify Targets

* In the Prometheus UI:

  * Click on **Status** → **Targets**
  * You should see: `Prometheus (1/1 up)`

---

## 📌 Next Steps

* Install **Node Exporter**
* Install **Grafana**
* Configure Prometheus to scrape Node Exporter
* Add Prometheus as a data source in Grafana

---

> 🧠 Tip: Keep your security group open for port `9090` (Prometheus), `3000` (Grafana), and `9100` (Node Exporter) only from trusted IPs.

---

Here's a concise `README.md` with only the **working steps** for configuring **Node Exporter** on your Linux (Ubuntu) server:

---

## 📘 18. Node Exporter Setup on Ubuntu (Systemd)


### ✅ Step-by-Step Instructions

#### 1. **Create a dedicated user**

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter
```

---

#### 2. **Download Node Exporter**

```bash
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar -xzf node_exporter-1.7.0.linux-amd64.tar.gz
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

---

#### 3. **Set permissions**

```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo chmod +x /usr/local/bin/node_exporter
```

---

#### 4. **Create systemd service**

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste the following content:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

---

#### 5. **Reload systemd & start service**

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

---

#### 6. **Check status**

```bash
sudo systemctl status node_exporter
```

Expected output:

```
Active: active (running)
```

---

#### 7. **Verify metrics endpoint**

Open in browser or curl:

```
http://<your-server-ip>:9100/metrics
```

---

Sure! Here’s a nicely formatted **README** section for the **Prometheus Plugin Integration** part you provided. You can add this to your project’s README.md file:

---

### 9: Configure Prometheus Plugin Integration

This section describes how to configure Prometheus to scrape metrics from **Node Exporter** and **Jenkins** in order to monitor the CI/CD pipeline and system metrics.

---

## Prerequisites

* Prometheus installed and running
* Node Exporter installed on the monitoring VM
* Jenkins with Prometheus plugin enabled for exposing metrics

---

## Steps to Configure Prometheus

### 1. Edit Prometheus Configuration

Open the Prometheus configuration file:

```bash
cd /etc/prometheus/
sudo vi prometheus.yml
```

Locate the default job called `prometheus`. At the end of the file, add the following job configurations:

```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets: ['<MonitoringVMip>:9100']

- job_name: 'jenkins'
  metrics_path: '/prometheus'
  static_configs:
    - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
```

> Replace `<MonitoringVMip>`, `<your-jenkins-ip>`, and `<your-jenkins-port>` with the appropriate IP addresses and ports for your setup.

Save and exit the editor (`esc` then `:wq`).

---

### 2. Validate Prometheus Configuration

Run the following command to check the configuration validity:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

If successful, you should see:

```
SUCCESS: /etc/prometheus/prometheus.yml is valid
```

---

### 3. Reload Prometheus Configuration

Reload the Prometheus configuration without restarting the service by running:

```bash
curl -X POST http://localhost:9090/-/reload
```

---

### 4. Open Firewall Ports (if needed)

Make sure port `9100` is open on your monitoring VM to allow Prometheus to scrape Node Exporter metrics.

---

### 5. Verify Targets in Prometheus UI

Access the Prometheus targets page in your browser:

```
http://<your-prometheus-ip>:9090/targets
```

You should see the following targets with status **UP**:

* `prometheus (1/1 up)`
* `node_exporter (1/1 up)`
* `jenkins (1/1 up)`

---

### 6. View Scraped Metrics

Click on **"show more"** next to the **jenkins** job and open the URL provided to see the metrics being scraped from Jenkins.

---


Here's a neatly formatted **README** section that walks through **Grafana Installation**, **Prometheus Integration**, and **Dashboard Setup for Node Exporter and Jenkins Monitoring**.

---

### 🖥️ 19. Grafana Monitoring Setup with Prometheus, Node Exporter & Jenkins

This guide walks you through installing Grafana OSS, integrating it with Prometheus, and setting up dashboards to monitor Node Exporter and Jenkins metrics.

---

## 📦 Step 1: Install Prerequisite Packages

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
```

---

## 🔑 Step 2: Import GPG Key for Grafana

```bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

---

## 📦 Step 3: Add Grafana Repository

### For Stable Releases:

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### For Beta Releases (optional):

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

---

## 🔄 Step 4: Update Package List

```bash
sudo apt-get update
```

---

## 🛠️ Step 5: Install Grafana OSS

```bash
sudo apt-get install grafana
```

---

## 🌐 Step 6: Access Grafana Web Interface

* Open Grafana in your browser:
  `http://<monitoring-server-ip>:3000`
* Default credentials:

  * **Username**: `admin`
  * **Password**: `admin`
* You can change the password or click **"Skip"** to continue.

---

## ⚙️ Step 7: Add Prometheus as a Data Source

1. In the Grafana UI, go to **Connections** → **Data Sources**.
2. Select **Prometheus**.
3. Enable the **Default** toggle.
4. In **HTTP → URL**, enter your Prometheus server URL (e.g., `http://localhost:9090`) **without the trailing slash**.
5. Scroll down and click **Save & Test**.
6. You should see a green checkmark if everything is configured correctly.

---

## 📊 Step 8: Import Node Exporter Dashboard

1. Open this URL in a browser:
   [https://grafana.com/grafana/dashboards/1860-node-exporter-full/](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)
2. Click **Copy ID to clipboard**.
3. In Grafana, click **+ (plus icon)** in the top-right corner → **Import Dashboard**.
4. Paste **`1860`** and click **Load**.
5. Under **Prometheus**, select your data source.
6. Click **Import** to load the dashboard.
7. Optionally, click **Save** (top-right corner).

---

## 📊 Step 9: Import Jenkins Dashboard

1. Open this URL in a browser:
   [https://grafana.com/grafana/dashboards/9964-jenkins-performance-and-health-overview/](https://grafana.com/grafana/dashboards/9964-jenkins-performance-and-health-overview/)
2. Click **Copy ID to clipboard**.
3. In Grafana, click **+ (plus icon)** in the top-right corner → **Import Dashboard**.
4. Paste **`9964`** and click **Load**.
5. Under **Prometheus**, select your data source.
6. Click **Import** to load the dashboard.
7. Optionally, click **Save** (top-right corner).

---

## ✅ Result

You now have Grafana set up with:

* Node Exporter Dashboard
* Jenkins CI/CD Performance Dashboard
* Prometheus as the data source

Monitor all your infrastructure and pipeline metrics from one place!

---

### 🚀 20. Deploy Application on AWS EKS Cluster using eksctl

This README will guide you through the process of deploying a Kubernetes (K8s) cluster using **Amazon EKS**, via `eksctl` on **VS Code / PowerShell**.

> ⚠️ **Notes before starting:**
> - Make sure you have the following tools installed: `eksctl`, `kubectl`, `awscli`
> - Run your terminal as **Administrator** (VS Code / PowerShell / CMD)
> - PowerShell does not support `\` as line continuation like Unix shells, so we will use **one-liners** or **backticks (`)**

---

## 🔧 Step 01: Create EKS Cluster (without Node Group)

### ✅ One-liner (Recommended in Windows/PowerShell)
```bash
eksctl create cluster --name=kastrocluster --region=ap-northeast-1 --zones=ap-northeast-1a,ap-northeast-1c --without-nodegroup
````

### ✅ Multi-line using Unix-style backslashes (`\`) – works in Linux/Mac

```bash
eksctl create cluster --name=kastrocluster \
                      --region=ap-northeast-1 \
                      --zones=ap-northeast-1a,ap-northeast-1c \
                      --without-nodegroup
```

### ✅ PowerShell-style (use backtick \` for line continuation)

```powershell
eksctl create cluster --name=kastrocluster `
                      --region=ap-northeast-1 `
                      --zones=ap-northeast-1a,ap-northeast-1c `
                      --without-nodegroup
```

> ⏳ This step takes \~20–25 mins. Watch CloudFormation stacks in the AWS Console.

---

## ✅ Verify Cluster

```bash
eksctl get cluster
```

Also check in AWS Console → **EKS Service** → `kastrocluster`

---

## 🔒 Step 02: Associate IAM OIDC Provider

IAM OIDC enables us to associate IAM roles with Kubernetes service accounts.

### ✅ One-liner

```bash
eksctl utils associate-iam-oidc-provider --region ap-northeast-1 --cluster kastrocluster --approve
```

### ✅ Multi-line (Unix/macOS)

```bash
eksctl utils associate-iam-oidc-provider \
    --region ap-northeast-1 \
    --cluster kastrocluster \
    --approve
```

### ✅ PowerShell

```powershell
eksctl utils associate-iam-oidc-provider `
    --region ap-northeast-1 `
    --cluster kastrocluster `
    --approve
```

---

## 🧱 Step 03: Create Node Group (Public Subnets)

### ✅ One-liner

```bash
eksctl create nodegroup --cluster=kastrocluster --region=ap-northeast-1 --name=kastrodemo-ng-public1 --node-type=t3.medium --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=Prajwal --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

### ✅ Multi-line (Unix/macOS)

```bash
eksctl create nodegroup --cluster=kastrocluster \
    --region=ap-northeast-1 \
    --name=kastrodemo-ng-public1 \
    --node-type=t3.medium \
    --nodes=2 \
    --nodes-min=2 \
    --nodes-max=4 \
    --node-volume-size=20 \
    --ssh-access \
    --ssh-public-key=Prajwal \
    --managed \
    --asg-access \
    --external-dns-access \
    --full-ecr-access \
    --appmesh-access \
    --alb-ingress-access
```

### ✅ PowerShell

```powershell
eksctl create nodegroup --cluster=kastrocluster `
    --region=ap-northeast-1 `
    --name=kastrodemo-ng-public1 `
    --node-type=t3.medium `
    --nodes=2 `
    --nodes-min=2 `
    --nodes-max=4 `
    --node-volume-size=20 `
    --ssh-access `
    --ssh-public-key=Prajwal `
    --managed `
    --asg-access `
    --external-dns-access `
    --full-ecr-access `
    --appmesh-access `
    --alb-ingress-access
```

### 🧾 Flag Explanations:

* `--ssh-access`: Enables SSH access to nodes
* `--ssh-public-key=Prajwal`: SSH key name in AWS Console
* `--managed`: Creates a managed node group
* `--asg-access`: Grants access to Auto Scaling Groups
* `--external-dns-access`: Allows use of external-dns with Route 53
* `--full-ecr-access`: Allows pulling images from Amazon ECR
* `--appmesh-access`: Grants access for AWS App Mesh
* `--alb-ingress-access`: Grants IAM permissions for AWS ALB Ingress Controller

---

## ✅ Step 04: Verify Cluster & Node Group

```bash
kubectl get nodes
```

In AWS Console → EKS → Cluster → **Compute Tab** → Node Group should be visible.

---

## 🧹 Optional Cleanup

### Step 05: Delete Node Group

```bash
eksctl get nodegroup --cluster=kastrocluster
eksctl delete nodegroup --cluster=kastrocluster --name=kastrodemo-ng-public1
```

### Step 06: Delete Cluster

```bash
eksctl delete cluster kastrocluster
```

---

## 🚀 Final: Deploy Your Application on EKS

```bash
kubectl apply -f your-app-deployment.yaml
```

Replace `your-app-deployment.yaml` with your actual deployment file.

---

## 🔗 References

* [eksctl GitHub](https://github.com/weaveworks/eksctl)
* [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)

---

> 📌 Make sure to keep your terminal open and don’t interrupt the process. Most issues arise due to permissions or closed terminals during provisioning.

```

