# DevOps Portfolio Project: Flask App CI/CD with Docker & Jenkins

Welcome to my hands-on DevOps project! This repository demonstrates how to build, test, containerize, and deploy a simple Flask application using Jenkins and Docker. The goal is to show end-to-end CI/CD pipeline skills that align with real-world platform engineering and DevOps roles.

---

## Project Overview
This project is designed to simulate a typical DevOps pipeline:

- Flask web app as a sample application
- Git-based source control
- Docker for containerization
- Jenkins for CI/CD automation
- Pipeline stages: build → test → dockerize → deploy (locally)

---

## Tech Stack
- Python (Flask)
- Git & GitHub
- Docker
- Jenkins
- Shell scripting (bash)
- Optionally: Docker Compose, Unit tests (pytest)

---

## Folder Structure
```bash
project-root/
├── app/
│   ├── app.py
│   └── requirements.txt
├── Jenkinsfile
├── Dockerfile
├── README.md
└── .gitignore
```

---

## Features Implemented
- [x] GitHub repository with clean commit history
- [x] Jenkins pipeline with multiple stages
- [x] Dockerfile to containerize the app
- [x] Jenkinsfile to automate build, test & deploy
- [x] Clear instructions to replicate on any system

---

## Step-by-Step Setup

### 1. Clone the Repo
```bash
git clone https://github.com/<one's-username>/flask-cicd-pipeline.git
cd flask-cicd-pipeline
```

### 2. Review the Flask App
`app/app.py` contains a basic Flask route.

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask + Jenkins Pipeline!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 3. Create a Dockerfile
```dockerfile
# Use official Python image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy app code
COPY app/ /app/

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Run the app
CMD ["python", "app.py"]
```

### 4. Jenkinsfile (Declarative Pipeline)
```groovy
pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-cicd-app"
    CONTAINER_NAME = "flask_app"
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker version' // Ensure Docker CLI is accessible
          sh "docker build -t $IMAGE_NAME ."
        }
      }
    }

    stage('Stop Existing Container') {
      steps {
        script {
          sh "docker ps -q --filter name=$CONTAINER_NAME | grep -q . && docker rm -f $CONTAINER_NAME || true"
        }
      }
    }

    stage('Run Docker Container') {
      steps {
        script {
          sh "docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME"
        }
      }
    }
  }

  post {
    success {
      echo ' Pipeline completed!'
    }
    failure {
      echo ' Pipeline failed!'
    }
  }
}

```

### 5. Push the Code to GitHub
Make sure to write a clean, human-readable commit history.

---

# How to Run the Flask App (Dockerized)

### What To Do Next in Ubuntu

Now that the project is uploaded to GitHub and ubuntu system has Docker and Jenkins installed, follow these steps to run and test everything locally:

## 1. Clone GitHub Repo (If not already done)

```bash
cd ~
git clone https://github.com/<one's-username>/flask-cicd-pipeline.git
cd flask-cicd-pipeline
```

## 2. (Optional) Test Flask App Without Docker

Make sure the app works standalone:

```bash
cd app
pip install -r requirements.txt
python app.py
```

Then visit [http://localhost:5000](http://localhost:5000) to confirm it runs. Use CTRL+C to stop the Flask app.

**Note:** `requirements.txt` file should be kept inside the `app/` directory, as that's where the `Dockerfile` and `Jenkinsfile` are configured to look for it when copying and installing dependencies.

## 3. Build Docker Image

From the root directory:

```bash
docker build -t flask-cicd-app .
```

## 4. Run Docker Container

```bash
docker run -d -p 5000:5000 --name flask_app flask-cicd-app
```

Visit: [http://localhost:5000](http://localhost:5000)

To stop the container:

```bash
docker stop flask_app
```

To remove it:

```bash
docker rm flask_app
```

## 5. Jenkins Setup Instructions (If Jenkins is Not Installed)

If Jenkins is not installed, follow these steps to set it up using Docker:

### Why We Create a Custom Jenkins Docker Image

By default, the official **jenkins/jenkins:lts** Docker image does not include the Docker CLI. This means commands like docker build and docker run will not work inside a Jenkins pipeline.

To fix this, we create a custom Jenkins image that:

* Installs Docker inside the Jenkins container

* Mounts the Docker socket

* Adds the jenkins user to the correct docker group (using host's GID)

This ensures Jenkins can access the host's Docker daemon securely and perform CI/CD operations.

### a. Create Jenkins Docker Image Directory and Dockerfile

```bash
mkdir ~/jenkins-docker && cd ~/jenkins-docker
nano Dockerfile
```

Paste the following into the Dockerfile:

```Dockerfile
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io && \
    groupdel docker || true && \
    groupadd -g 982 docker && \
    usermod -aG docker jenkins

USER jenkins
```

To find uder's host Docker group ID (GID):

```bash
getent group docker
```

Use the GID (e.g., 982) in the Dockerfile `groupadd` line.

### b. Build the Custom Jenkins Image

```bash
docker build --build-arg DOCKER_GID=982 -t my-jenkins-docker .
```

### c. Stop & Remove Existing Jenkins (If Any)

```bash
docker stop jenkins
docker rm jenkins
```

### d. Run Jenkins Container with Docker Socket Mounted

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  my-jenkins-docker
```

> We use this custom image instead of the default `jenkins/jenkins:lts` because it includes the Docker CLI and mounts the Docker socket. This setup allows Jenkins (running as a non-root user) to access and execute Docker commands without permission issues.

To verify inside the container:

```bash
docker exec -u root -it jenkins bash
id jenkins
ls -l /var/run/docker.sock
```

Expected:

* Jenkins user in `docker` group
* Docker socket group ownership matching GID (e.g., 982)

Exit the container:

```bash
exit
```

### e. Test Docker from Jenkins

```bash
docker exec -it jenkins docker version
```

User should now see Docker Client and Server versions with no permission errors.

---

## 6. Restarting Jenkins After Reboot (If User Had Jenkins Before)

If user already installed Jenkins previously (yesterday), and user has restarted Ubuntu:

### a. First, ensure Docker is running (usually starts automatically)

```bash
sudo systemctl start docker
```

### b. Then start Jenkins container again:

```bash
docker start jenkins
```

To check that it’s running:

```bash
docker ps
```

Now Jenkins should be live again at:

```
http://localhost:8080
```

And the Flask app (if user runs its container) will be at:

```
http://localhost:5000
```
---

## 7. Run Jenkins Job with Pipeline

1. Open Jenkins in browser: [http://localhost:8080](http://localhost:8080)
2. Click on "New Item"
3. Enter a name for one's job, select "Pipeline", and click OK
4. Scroll down to the **Pipeline** section
5. Under **Definition**, choose `Pipeline script from SCM`
6. For **SCM**, select `Git`
7. In the **Repository URL** field, enter user's GitHub repo URL:

   ```
   https://github.com/<one's-username>/flask-cicd-pipeline.git
   ```
8. Leave credentials blank if the repo is public, or add credentials if it's private
9. Under **Branch Specifier**, enter:

   ```
   */main
   ```
10. Click "Save"
11. Click **"Build Now"** to trigger the pipeline

This will:

* Clone your GitHub repository
* Read the Jenkinsfile
* Build the Docker image
* Run Flask container automatically

---

## Done!

The CI/CD pipeline is now working locally using Docker and Jenkins.

---

User should see:
> "Hello from Flask + Jenkins Pipeline!"

---

## 8. Reboot Recovery Quick Commands

When starting user's system next time (after reboot), run the following to restore both apps:

```bash
# Start Docker daemon (if needed)
sudo systemctl start docker

# Start Jenkins container
docker start jenkins

# Start Flask app container (optional)
docker start flask_app
```

Then open in browser:

* Jenkins: [http://localhost:8080](http://localhost:8080)
* Flask App: [http://localhost:5000](http://localhost:5000)

# Jenkins + Docker: Full Debugging & Fix Guide

This guide documents the **complete troubleshooting journey** to fix the `docker: permission denied` error when using Docker inside a Jenkins container.
It includes explanations of why each step was necessary — so user not only fixes the problem, but **understands it deeply**.

---

## The Problem

When running a Jenkins pipeline that builds a Docker image, the following errors appeared:

```bash
+ docker version
/var/jenkins_home/.../script.sh.copy: 1: docker: not found
```

After installing Docker inside Jenkins, the error changed to:

```bash
permission denied while trying to connect to the Docker daemon socket
```

---

## Why This Happens: Docker + Jenkins Theory

### Why `docker` must be **mounted** into Jenkins:

Docker runs as a **daemon (background service)** on user's host machine. Containers, including Jenkins, are isolated and **cannot see the Docker daemon unless user explicitly shares access**.

So, this line is essential:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

It mounts the host's Docker socket into the Jenkins container so that Jenkins can issue Docker commands.

### Why Access Was Denied

Even with the socket mounted, Jenkins runs as a **non-root user** (named `jenkins`). If this user isn’t part of the group that owns the Docker socket (`docker` group on the host), it **cannot access** Docker.

### Why We Created a Custom Image

The default `jenkins/jenkins:lts` image:

* Does **not** come with Docker installed
* Does **not** have permissions pre-configured to access the host Docker daemon

So we created a custom image that:

* Installs Docker inside the container 
* Adds the `jenkins` user to the **correct `docker` group** 

---

## Find Host Docker Group ID (GID)

### Step 1: Get user's host Docker GID

From user's **host terminal** (not inside a container):

```bash
getent group docker
```

### Example Output:

```
docker:x:982:rubaiya
```

Here, `982` is the GID we must use **inside Jenkins**.

---

## Step-by-Step Fix

### Step 1: Create Custom Dockerfile

```Dockerfile
FROM jenkins/jenkins:lts

USER root

# Remove default docker group and recreate with correct GID from host (e.g. 982)
RUN apt-get update && \
    apt-get install -y docker.io && \
    groupdel docker && \
    groupadd -g 982 docker && \
    usermod -aG docker jenkins

USER jenkins
```

### Step 2: Build Jenkins Image

```bash
cd ~/jenkins-docker

# Replace 982 with user's actual GID if different
docker build -t my-jenkins-docker .
```

### Step 3: Stop & Remove Old Jenkins Container

```bash
docker stop jenkins
docker rm jenkins
```

### Step 4: Run Jenkins with Docker Socket Mounted

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  my-jenkins-docker
```

---

## Verification & Debugging

### Inside Container (as root):

```bash
docker exec -u root -it jenkins bash
```

Check socket permissions:

```bash
ls -l /var/run/docker.sock
```

Expected:

```
srw-rw---- 1 root 982 /var/run/docker.sock
```

Check Jenkins user groups:

```bash
id jenkins
```

Expected:

```
groups=1000(jenkins),982(docker)
```

If not:

```bash
usermod -aG docker jenkins
exit
```

Then:

```bash
docker restart jenkins
```

### Final test:

```bash
docker exec -it jenkins docker version
```

User should now see both **Client** and **Server** Docker version outputs with **no permission errors**.

---

## Useful Debugging Commands

```bash
# On host
getent group docker

# Inside container
ls -l /var/run/docker.sock
id jenkins
```

---

## Final Outcome

* Jenkins container has Docker CLI 
* Jenkins user has permission to access host Docker 
* Docker socket is properly mounted 
* Jenkins pipeline now successfully builds and runs Docker containers! 

---

## Why This Project Matters for DevOps domain
- Shows one understands **CI/CD automation**
- Demonstrates use of **Docker and Jenkins** (core tools in DevOps)
- Real, working example of **pipeline as code**
- Good base for expansion: testing, logging, monitoring, Kubernetes

---

## What’s Next (Project Ideas for Portfolio Expansion)
- Add **unit tests** and integrate **pytest** into the pipeline
- Use **Docker Compose** to simulate multi-service apps
- Add **Nginx reverse proxy** or **SSL support**
- Set up **GitHub Webhooks** to trigger Jenkins build on push
- Host the Jenkins server on **cloud VM** (AWS/GCP)
- Extend to **Kubernetes deployment** using Minikube or kind

---

## Contribution & Contact
This is a personal portfolio project created as part of my journey toward becoming a DevOps/Platform Engineer. Feel free to fork it, use it, or reach out with suggestions.

---

> *Built from scratch, with a lot of curiosity and a clear goal: becoming a DevOps Engineer who automates everything but never stops learning.*
