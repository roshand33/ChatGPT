
# 🚀 **Deploy Chat-GPT Clone App on Docker using Jenkins – DevSecOps Project**

<div align="center">

<img src="https://github.com/user-attachments/assets/17b4f0e7-98fa-4c35-9b50-b88b7c5f0b6d" width="180"/>

<br><br>

<img src="https://github.com/user-attachments/assets/2f3e07f8-56de-435b-a68d-6f30c2a865ed" width="100%" />
<p><b>ChatGPT Clone – Home Page</b></p>
</div>

---

# 📌 **Project Overview**

In this project, we deploy a **ChatGPT-Clone App** using:

* **Jenkins CI/CD**
* **SonarQube (code quality)**
* **Trivy (security scanning)**
* **Docker (build + deploy)**



# ✅ **STEP 1 — Launch EC2 Instance**

* AMI: **Ubuntu 22.04**
* Instance Type: **t2.medium**
* Storage: **30 GB**
* Security Group:

  * 22 (SSH)
  * 80 / 8000 (App)
  * 8080 (Jenkins)
  * 9000 (SonarQube)

---

# ✅ **STEP 2 — Install Required Tools (Jenkins, Docker, SonarQube, Trivy)**

## 🟦 **Install Jenkins**

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```

## 🟩 **Install Docker**

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

## 🟧 **Install SonarQube**

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## 🟥 **Install Trivy**

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | \
sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install trivy -y
```

---

# ✅ **STEP 3 — Login to Jenkins**

Visit:

```
http://<EC2-IP>:8080
```

Unlock Jenkins → Install Suggested Plugins.

---

# ✅ **STEP 4 — Configure SonarQube**

Login:

```
username: admin
password: admin
```

Generate Token:
**Admin → My Account → Security → Generate Token**

Save token as: **sonar-token**

---

# ✅ **STEP 5 — Add Credentials in Jenkins**

Go to:
**Manage Jenkins → Credentials → Global → Add Credentials**

### Add:

1️⃣ **Git Credentials**
2️⃣ **DockerHub Credentials** → *ID:* `docker-cred`
3️⃣ **SonarQube Token** → *ID:* `sonar-token`

---

# ✅ **STEP 6 — Install Required Jenkins Plugins**

Install:

```
Docker
SonarQube Scanner
Eclipse Temurin Installer
Stage View
```

---

# ✅ **STEP 7 — Add Tools (Manage Jenkins → Tools)**

| Tool              | Name          | Version |
| ----------------- | ------------- | ------- |
| JDK               | jdk17         | 17      |
| SonarQube Scanner | sonar-scanner | latest  |
| Docker            | docker        | system  |

---

# ✅ **STEP 8 — Create Jenkins Pipeline**

Create a new **Pipeline Job** → Paste below:

---

# 🚀 **Final Jenkins Pipeline for ChatGPT Clone Deployment (Docker)**

```groovy
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE = "abhipraydh96/chatgpt-fastapi"
        APP_NAME = "chatgpt-app"
    }

    stages {

        stage('Pull Code') {
            steps {
                git branch: 'main', url: 'https://github.com/abhipraydhoble/doesntmatter.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=ChatGPT \
                    -Dsonar.projectKey=ChatGPT
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivy-fs-scan.txt"
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh "docker build -t ${IMAGE} ."
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker push ${IMAGE}"
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ${IMAGE} > trivy-image.txt"
            }
        }

        stage('Deploy Container Locally') {
            steps {
                sh """
                docker stop ${APP_NAME} || true
                docker rm ${APP_NAME} || true
                docker run -d --name ${APP_NAME} -p 8000:8000 ${IMAGE}
                """
            }
        }
    }
}
```

---

# 🚨 **Permissions Fix Before Running Pipeline**

```bash
sudo usermod -aG docker jenkins
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

---

# 🎉 **Output Screenshots**

These are similar to what you’ll get:

### 🔹 Trivy Scan Output

*(example)* <img width="100%" src="https://github.com/user-attachments/assets/ba9b2a7e-0e58-448f-9f24-4a00e5d1efd0"/>

### 🔹 SonarQube Report

<img width="100%" src="https://github.com/user-attachments/assets/df9e8a1e-f903-4be8-8d5a-97af04f16ca4"/>

### 🔹 Running ChatGPT App 🍀

Visit:

```
http://<EC2-IP>:8000
```

<img width="100%" src="https://github.com/user-attachments/assets/fe5fbe19-e62f-42c7-b3e7-9454bc1cec5b"/>

---


