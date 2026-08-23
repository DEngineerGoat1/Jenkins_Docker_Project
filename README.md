# Jenkins Docker CI/CD Project

## Project Overview

This project was completed as part of my **Week 6 – Day 2 Jenkins assignment**. The purpose of the project was to gain hands-on experience using **Jenkins, Docker, GitHub, and AWS EC2** to create and automate a basic CI/CD pipeline.

During this project, I launched an Ubuntu EC2 instance, installed Docker, ran Jenkins inside a Docker container, created a Jenkins pipeline, connected Jenkins to GitHub, and automated the creation of a Docker image.

---

## Technologies Used

* AWS EC2
* Ubuntu Linux
* Jenkins
* Docker
* Git
* GitHub
* VS Code
* Python

---

## Project Files

```text
Jenkins_Docker_Project/
├── Dockerfile
├── Jenkinsfile
├── app.py
└── README.md
```

### `app.py`

A simple Python application used as the application for the Docker image.

### `Dockerfile`

Contains the instructions Docker uses to package the Python application into a Docker image.

### `Jenkinsfile`

Defines the Jenkins CI/CD pipeline and its automation stages.

---

# Project Architecture

```text
Local VS Code
      |
      v
    GitHub
      |
      v
    Jenkins
      |
      v
    Docker
      |
      v
 Docker Image
```

My project files were pushed from my local computer to GitHub. Jenkins retrieved the project from GitHub and used the Jenkinsfile to automatically execute the pipeline.

---

# Jenkins Pipeline

The Jenkins pipeline included three stages:

## 1. Build

The Build stage automatically created the Docker image.

```bash
docker build -t dcegoat1/jenkins:latest .
```

## 2. Test

The Test stage represented testing the application before deployment.

## 3. Deploy

The Deploy stage represented the deployment portion of the CI/CD process.

For this learning project, the Test and Deploy stages used basic `echo` commands while the Build stage performed the actual Docker build.

---

# AWS EC2 Setup

I launched an Ubuntu EC2 instance and configured the required Security Group access.

Ports used during the project included:

| Port  | Purpose                     |
| ----- | --------------------------- |
| 22    | SSH access to EC2           |
| 8080  | Jenkins web interface       |
| 50000 | Jenkins agent communication |

I connected to the EC2 instance through SSH from my VS Code terminal.

Example:

```bash
ssh -i "key-name.pem" ubuntu@EC2-PUBLIC-IP
```

---

# Docker Setup

Docker was installed and verified on the EC2 instance.

Verification command:

```bash
docker --version
```

I also tested Docker using:

```bash
sudo docker run hello-world
```

---

# Running Jenkins with Docker

Jenkins was initially launched using the Jenkins LTS Docker image:

```bash
sudo docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  jenkins/jenkins:lts
```

I then accessed Jenkins through the EC2 public IP on port `8080`.

```text
http://EC2-PUBLIC-IP:8080
```

---

# GitHub Integration

The local project was initialized as a Git repository.

```bash
git init
git add .
git commit -m "Add Jenkins Docker project"
git branch -M main
git push -u origin main
```

Jenkins was configured to use:

**Pipeline script from SCM → Git**

The pipeline retrieved the `Jenkinsfile` directly from the GitHub repository.

---

# Docker Automation with Jenkins

One of the main goals of this project was allowing Jenkins to automatically build a Docker image.

Jenkins successfully executed:

```bash
docker build -t dcegoat1/jenkins:latest .
```

The Jenkins Console Output confirmed that the Docker image was successfully created and the pipeline completed with:

```text
Finished: SUCCESS
```

---

# Troubleshooting & Lessons Learned

This project required several troubleshooting steps.

### Jenkins Could Not Access Docker

Jenkins was running inside a Docker container and initially could not execute Docker commands.

I learned that Docker being installed on the EC2 host does not automatically mean a Jenkins container can access Docker.

I configured Jenkins so it could communicate with the EC2 host's Docker engine using the Docker socket.

---

### Docker Socket Permission Issue

Jenkins initially received a permission error when attempting to access:

```text
/var/run/docker.sock
```

I troubleshot the Docker socket permissions and configured my learning environment so Jenkins could communicate with the Docker engine.

---

### Jenkins Container Stopped — Exit 137

At one point the Jenkins container stopped with:

```text
Exited (137)
```

The EC2 instance had limited memory, so I added swap space to provide additional memory protection.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

### Jenkins Node Went Offline

Jenkins later reported:

```text
Waiting for next available executor
```

After troubleshooting, I discovered that Jenkins had taken the Built-In Node offline because the EC2 instance was running extremely low on disk space.

The original EBS volume was only:

```text
8 GiB
```

I increased the EBS volume to:

```text
20 GiB
```

Then expanded the Linux partition:

```bash
sudo growpart /dev/nvme0n1 1
```

And expanded the filesystem:

```bash
sudo resize2fs /dev/nvme0n1p1
```

Afterward, the server had approximately **13 GB of available disk space**, Jenkins came back online, and the pipeline successfully completed.

---

# Final Result

The final automated workflow was:

```text
GitHub Repository
       |
       v
Jenkins reads Jenkinsfile
       |
       v
Build Docker Image
       |
       v
Test Stage
       |
       v
Deploy Stage
       |
       v
SUCCESS
```

The project successfully demonstrated how **AWS, GitHub, Jenkins, and Docker can work together in a basic CI/CD workflow**.

---

## What I Learned

This project helped me better understand that Jenkins is more than just an automation tool. It acts as the connection between source code and the different steps required to build, test, and deploy an application.

I also learned that troubleshooting is an important part of DevOps. I worked through Docker permissions, container memory limitations, Jenkins executors, disk-space problems, GitHub integration, and EBS storage expansion before successfully completing the automated Docker build.

The biggest takeaway was seeing how several tools I had been learning separately could finally work together as one automated cloud engineering workflow.
