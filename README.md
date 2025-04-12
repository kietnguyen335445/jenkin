# Jenkins Pipeline for Java Application

This project demonstrates a CI/CD pipeline for building and deploying a Java-based Spring Boot application using tools such as Jenkins, Maven, SonarQube, Docker, ArgoCD, and Kubernetes.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Setup Instructions](#setup-instructions)
   - [AWS EC2 Instance](#setup-an-aws-ec2-instance)
   - [Run Java Application on EC2](#run-java-application-on-ec2)
3. [Continuous Integration](#continuous-integration)
   - [Install and Setup Jenkins](#install-and-setup-jenkins)
   - [Jenkins Pipeline Creation](#create-a-new-jenkins-pipeline)
   - [SonarQube Configuration](#configure-a-sonar-server-locally)
4. [Continuous Delivery](#continuous-delivery)
   - [ArgoCD Setup](#argo-cd-setup)
   - [Minikube and Kubernetes](#setup-minikube)
5. [References](#references)

---

## Project Overview
The project integrates various DevOps tools to streamline the development process, improve code quality, and enable automated deployment:
- **Version Control**: GitHub is used to manage the source code and Dockerfiles.
- **Continuous Integration**: Jenkins builds the application, runs tests, and analyzes code quality using SonarQube.
- **Containerization**: Docker is used to containerize the application, and images are pushed to DockerHub.
- **Continuous Delivery**: ArgoCD automates Kubernetes-based deployments, following the GitOps approach.

---

## Setup Instructions

### Setup an AWS EC2 Instance
1. Login to AWS and navigate to the EC2 Console.
2. Launch an instance:
   - **Name**: `Main-Server`
   - **AMI**: Ubuntu
   - **Instance Type**: `t2.medium`
   - **Security Group**: Allow SSH, HTTP, and HTTPS traffic.
3. Create a key pair for secure access.

---

### Run Java Application on EC2
1. Clone the repository:
   ```bash
   git clone https://github.com/sunitabachhav2007/Jenkins-Zero-To-Hero.git
   cd Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/spring-boot-app
2. Install dependencies
  sudo apt update
  sudo apt install maven docker.io

3. Build and run the application:
   mvn clean package
   docker build -t ultimate-cicd-pipeline:v1 .
   docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
   
5. Access the application: http://<EC2_PUBLIC_IP>:8010

Continuous Integration

Install and Setup Jenkins
  1. Install Jenkins:
     ```bash
     sudo apt update
     sudo apt install openjdk-11-jre
     curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
     sudo apt-get update
     sudo apt-get install jenkins
     ```
  2. Access Jenkins at http://<EC2_PUBLIC_IP>:8080 and complete the setup.

Create a New Jenkins Pipeline
  1. Install plugins: Docker Pipeline, SonarQube Scanner.
  2. Configure credentials for SonarQube, DockerHub, and GitHub.
  3. Set up the pipeline using the JenkinsFile

Configure a Sonar Server
  1. Install SonarQube
       ```bash
           wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
           unzip sonarqube-9.4.0.54424.zip
           chmod -R 755 sonarqube-9.4.0.54424
          ./sonarqube-9.4.0.54424/bin/linux-x86-64/sonar.sh start
       ```
  2. Access SonarQube at http://<EC2_PUBLIC_IP>:9000

Continuous Delivery
Argo CD Setup
  1. Install Minikube
       ```bash
                     curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
                     sudo install minikube-linux-amd64 /usr/local/bin/minikube
                     minikube start --driver=docker                  
       ```
  2. Install Argocd
     ```bash
           kubectl create namespace argocd
           kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
     
