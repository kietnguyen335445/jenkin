# ☕ Jenkins Pipeline for basic Java Application with Maven, SonarQube, Argo CD and Kubernetes

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

Select AMIs as ```Ubuntu``` and select Instance Type as ```t2.medium```. Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.
![ec2](https://github.com/user-attachments/assets/9bb74427-512f-4a77-b3d8-dde92bf4bf86)


