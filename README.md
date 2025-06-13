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
git clone https://github.com/<your-username>/flask-cicd-pipeline.git
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
FROM python:3.10-slim
WORKDIR /app
COPY app/ /app/
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

### 4. Jenkinsfile (Declarative Pipeline)
```groovy
pipeline {
  agent any
  stages {
    stage('Clone') {
      steps {
        git 'https://github.com/<your-username>/flask-cicd-pipeline.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t flask-cicd-app .'
      }
    }
    stage('Run Container') {
      steps {
        sh 'docker run -d -p 5000:5000 --name flask_app flask-cicd-app'
      }
    }
  }
}
```

### 5. Push Your Code to GitHub
Make sure to write a clean, human-readable commit history.

---

## Final Result
Once deployed, open your browser:
```
http://localhost:5000
```
You should see:
> "Hello from Flask + Jenkins Pipeline!"

---

## Why This Project Matters for DevOps Jobs
- Shows you understand **CI/CD automation**
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
