# ☕ Jenkins Pipeline for Java Application with Maven, SonarQube, Argo CD and Kubernetes

This project demonstrates a complete CI/CD pipeline for a Java-based application using:
- **Jenkins** for automation
- **Maven** for build and dependency management
- **SonarQube** for static code analysis
- **Argo CD** for GitOps-based deployment
- **Kubernetes** for container orchestration

---

## 📚 Table of Contents

- [📚 Table of Contents](#-table-of-contents)
- [📌 Features](#-features)
- [🧱 Architecture](#-architecture)
- [🛠️ Technologies Used](#️-technologies-used)
- [📁 Project Structure](#-project-structure)
- [🔧 Jenkins Pipeline Stages](#-jenkins-pipeline-stages)
- [🚀 Setup Instructions](#-setup-instructions)
- [✅ How to Run](#-how-to-run)
- [📸 Screenshots](#-screenshots)
- [🙌 Contributing](#-contributing)
- [📄 License](#-license)

---

## 📌 Features

- Automated CI/CD using Jenkins
- Code quality checks using SonarQube
- Maven-based build system
- Dockerized Java app with deployment to Kubernetes
- GitOps-based CD using Argo CD

---

## 🧱 Architecture

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

## 📁 Project Structure

```plaintext
.
├── Jenkinsfile
├── src/
├── pom.xml
├── sonar-project.properties
├── manifests/                 # K8s YAML for deployment/service
│   └── deployment.yaml
├── Dockerfile
└── README.md

