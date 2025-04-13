# ☕ Jenkins Pipeline for a simple Java Application with Maven, SonarQube, Argo CD and Kubernetes

This project demonstrates a complete CI/CD pipeline for a Java-based application using:
- **Jenkins** for automation
- **Maven** for build and dependency management
- **SonarQube** for static code analysis
- **Argo CD** for GitOps-based deployment
- **Kubernetes** for container orchestration

---

# 📚 Table of Contents

- [📚 Table of Contents](#-table-of-contents)
- [📌 Features](#-features)
- [🧱 Architecture](#-architecture)
- [🛠️ Technologies Used](#️-technologies-used)
- [🔧 Jenkins Pipeline Stages](#-jenkins-pipeline-stages)
- [🚀 Setup Instructions](#-setup-instructions)
- [✅ How to Run](#-how-to-run)
- [📸 Screenshots](#-screenshots)
- [🙌 Contributing](#-contributing)
- [📄 License](#-license)

---

# 📌 Features

- Automated CI/CD using Jenkins
- Code quality checks using SonarQube
- Maven-based build system
- Dockerized Java app with deployment to Kubernetes
- GitOps-based CD using Argo CD

---

# 🧱 Architecture
---


## 🛠️ Technologies Used

- **Java 17**
- **Maven**
- **Jenkins**
- **SonarQube**
- **Docker**
- **Argo CD**
- **Kubernetes (K8s)**
- **GitHub/GitLab**

---
## 🔧 Jenkins Pipeline Stages
```bash
pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/kietnguyen335445/jenkin.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd spring-boot-app && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://54.151.200.64:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'Sonar', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "tolitun/jenkin-argo-cicd-pipeline:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerhub')
      }
      steps {
        script {
            sh 'cd spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "jenkin"
            GIT_USER_NAME = "kietnguyen335445"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "tkiet3093@gmail.com"
                    git config user.name "Kiet Nguyen"
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    if ! git diff --cached --quiet; then
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    else
                        echo "No changes to commit"
                    fi
                '''
            }
        }
    }
  }
}

```
---
#  🚀 Setup Instructions
## Setup an AWS EC2 Instance
Login to an AWS account using a user with admin privileges and ensure your region is set to ``` ap-souteast-1``` ``` Singapore ```
Move to the EC2 console. Click ```Launch Instance.```

For name use ```Main-Server```

Select AMIs as ```Ubuntu``` and select Instance Type as ```t2.medium```. 
Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.

![ec2](https://github.com/user-attachments/assets/9bb74427-512f-4a77-b3d8-dde92bf4bf86)

## Run Java application on EC2
```bash
git clone https://github.com/kietnguyen335445/jenkin
cd spring-boot-app
sudo apt update
sudo apt install maven
mvn clean package
mvn -v
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
docker build -t jenkin-argo-cicd-pipeline:v1 .
docker run -d -p 8010:8080 -t jenkin-argo-cicd-pipeline:v1
```
## Continuous Integration
Step 1: Install Jenkins
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
Step 2: Setup Jenkins
You can get the ec2-instance-public-ip-address from your AWS EC2 console page.

Edit the inbound traffic rule to only allow custom TCP port ```8080```

http://:<ec2-instance-public-ip-address>8080
```sudo cat /var/lib/jenkins/secrets/initialAdminPassword``` to get the password of jenkin
## Create a new Jenkins pipeline
Click on New Item. Select Pipeline and Enter an Item name.
![jenkin-2](https://github.com/user-attachments/assets/5813f77a-ff3f-45cb-9654-d75e72390f20)
## Install the necessary Jenkins plugins
Goto Jenkins Dashboard \==> Manage Jenkins \==> Plugins \==> Available plugins

Docker Pipeline

SonarQube Scanner
## Configure a Sonar Server
```bash
sudo adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
sudo apt install unzip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```
Edit the inbound traffic rule to only allow custom TCP port ```9000```
Username: admin
Password: admin
# Create Credentials in Jenkins
Step 1: Credential for SonarQube
Go to My Account ==> Security
## On jenkins
Go to -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials
![jenkin5](https://github.com/user-attachments/assets/7ab44197-25e7-4b30-9ba2-88212a025451)
Step 2: Create DockerHub Credential in Jenkins
Goto -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials
![jenkin6](https://github.com/user-attachments/assets/5b3ab05a-a1d4-43ea-9674-be2318bd01ee)
Step 3: Create GitHub credential in Jenkins
Goto GitHub ==> Setting ==> Developer Settings ==> Personal access tokens ==> Tokens(Classic) ==> Generate new token
![jenkin7](https://github.com/user-attachments/assets/f37e3c47-8209-4d54-841e-905230eb27de)
# Install Docker
```bash
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
sudo systemctl restart docker
```
Do jenkin Restart
# Run Pipeline
![jenkin-pip](https://github.com/user-attachments/assets/4319ebcf-9992-4085-a61b-99de2198ea5f)
SonarQube Output:
![sonar](https://github.com/user-attachments/assets/242945a5-3310-4211-9ca3-f80f883083a5)
Docker Hub:
![docker](https://github.com/user-attachments/assets/157fd359-3ff4-4726-a934-203bb044d8d0)
# Continuous Delivery Part
# Argo CD Setup
Install on Ubuntu locally
# Setup Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo usermod -aG docker $USER && newgrp docker
sudo reboot docker
minikube start --driver=docker
```
# Install kubectl
```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version
```
# Install Argo CD operator
```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.24.0/install.sh | bash -s v0.24.0
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
kubectl get csv -n operators
kubectl get pods -n operators
```
```kubectl edit svc example-argocd-server``` and change from ClusterIP to NodePort
![argocd-nodeport](https://github.com/user-attachments/assets/8b1ceedc-64e4-4b2c-b477-7af27c43407f)
# Password for Argo CD
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Setup Github Repository manifest and Kubernetes cluster.
![argocd-2](https://github.com/user-attachments/assets/d451f2a9-ed56-459b-819c-29fe5e9f41d3)

![argocd-3](https://github.com/user-attachments/assets/189f9963-6b80-4ec7-932d-0349ac6ecf38)

![argocd](https://github.com/user-attachments/assets/329fd774-1b47-4b3c-a3d7-139ac6707ddd)








