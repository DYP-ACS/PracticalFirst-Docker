# 🐳 My First Docker Application

A beginner-friendly tutorial to learn Docker by containerizing a simple HTML website using NGINX.

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Step-by-Step Guide](#step-by-step-guide)
- [Understanding Docker Concepts](#understanding-docker-concepts)
- [Container Internals](#container-internals)
- [AWS ECR Deployment with GitHub Actions](#aws-ecr-deployment-with-github-actions)
- [Cleanup](#cleanup)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This project demonstrates how to:
- Create a simple HTML webpage
- Containerize it using Docker
- Serve it using NGINX web server
- Understand port mapping and container management

**What you'll learn:**
- Docker basics
- Creating Dockerfiles
- Building Docker images
- Running containers
- Port mapping
- Container lifecycle management

## ✅ Prerequisites

Before starting, ensure you have:
- Docker Desktop installed ([Download here](https://www.docker.com/products/docker-desktop))
- Basic understanding of HTML
- Terminal/Command Prompt access

**Verify Docker installation:**
```bash
docker --version
```

## 📁 Project Structure

```
PracticalFirst-Docker/
│
├── .github/
│   └── workflows/
│       └── deploy.yml  # GitHub Actions workflow for AWS ECR
├── index.html          # Your HTML webpage
├── Dockerfile          # Docker configuration
└── README.md           # This file
```

## 🚀 Step-by-Step Guide

### Step 1: Create the HTML File

Create a file named `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First Docker App</title>
</head>
<body>
    <h1>Hello Students 🚀</h1>
    <p>This HTML page is running inside Docker!</p>
</body>
</html>
```

### Step 2: Create the Dockerfile

Create a file named `Dockerfile` (no extension):

```dockerfile
# Step 1: Use official nginx base image
FROM nginx:alpine

# Step 2: Remove default nginx html
RUN rm -rf /usr/share/nginx/html/*

# Step 3: Copy our HTML file
COPY index.html /usr/share/nginx/html/

# Step 4: Expose port 80
EXPOSE 80

# Step 5: Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Step 3: Build the Docker Image

Build your Docker image:

```bash
docker build -t my-html-app .
```

**Command breakdown:**
- `docker build` - Build a Docker image
- `-t my-html-app` - Tag/name the image as "my-html-app"
- `.` - Use current directory as build context

**Verify the image was created:**
```bash
docker images
```

You should see `my-html-app` in the list.

### Step 4: Run the Container

Start your container:

```bash
docker run -d -p 8080:80 --name html-container my-html-app
```

**Command breakdown:**
- `docker run` - Create and start a container
- `-d` - Run in detached mode (background)
- `-p 8080:80` - Map port 8080 (host) to port 80 (container)
- `--name html-container` - Name the container "html-container"
- `my-html-app` - Use the image we built

**Verify the container is running:**
```bash
docker ps
```

### Step 5: View Your Application

Open your browser and navigate to:

```
http://localhost:8080
```

🎉 You should see:
```
Hello Students 🚀
This HTML page is running inside Docker!
```

## 🧠 Understanding Docker Concepts

### Dockerfile Instructions Explained

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image to build upon | `FROM nginx:alpine` - Uses lightweight NGINX |
| `RUN` | Execute commands during build | `RUN rm -rf /usr/share/nginx/html/*` |
| `COPY` | Copy files from host to container | `COPY index.html /usr/share/nginx/html/` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 80` |
| `CMD` | Default command to run when container starts | `CMD ["nginx", "-g", "daemon off;"]` |

### Why nginx:alpine?

- **NGINX**: High-performance web server
- **alpine**: Minimal Linux distribution (~5MB vs ~100MB for Ubuntu)
- **Result**: Fast, lightweight container

### Port Mapping Explained

When you use `-p 8080:80`:
- **8080**: Port on your local machine (host)
- **80**: Port inside the container
- Traffic to `localhost:8080` → forwards to container port 80

## 🏗 Container Internals

Understanding the flow:

```
┌─────────────────────────────────────────┐
│         Your Computer (Host)            │
│                                         │
│  Browser → http://localhost:8080       │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │   Port Mapping       │
        │   8080 → 80          │
        └──────────┬───────────┘
                   │
                   ▼
┌──────────────────────────────────────────┐
│         Docker Container                 │
│                                          │
│  ┌────────────────────────────────┐     │
│  │  NGINX Web Server (Port 80)    │     │
│  │                                 │     │
│  │  Serves: index.html             │     │
│  └────────────────────────────────┘     │
│                                          │
└──────────────────────────────────────────┘
```

**Flow:**
1. Browser requests → `localhost:8080`
2. Docker maps → Container port `80`
3. NGINX receives request
4. ☁️ AWS ECR Deployment with GitHub Actions

This section covers how to automatically build and push your Docker image to AWS Elastic Container Registry (ECR) using GitHub Actions.

### Prerequisites for AWS ECR

1. **AWS Account** - [Sign up here](https://aws.amazon.com/)
2. **AWS CLI installed** (optional, for local testing)
3. **GitHub repository** for your project

### Step 1: Create AWS ECR Repository

#### Option A: Using AWS Console

1. Go to [AWS ECR Console](https://console.aws.amazon.com/ecr/)
2. Click **Create repository**
3. Set repository name: `my-html-app`
4. Leave settings as default
5. Click **Create repository**

#### Option B: Using AWS CLI

```bash
aws ecr create-repository \
    --repository-name my-html-app \
    --region us-east-1
```

### Step 2: Create AWS IAM User for GitHub Actions

1. Go to [AWS IAM Console](https://console.aws.amazon.com/iam/)
2. Click **Users** → **Add users**
3. Username: `github-actions-ecr`
4. Select **Access key - Programmatic access**
5. Click **Next: Permissions**
6. Click **Attach existing policies directly**
7. Search and select: `AmazonEC2ContainerRegistryPowerUser`
8. Click **Next** → **Create user**
9. **IMPORTANT**: Save the **Access Key ID** and **Secret Access Key**

### Step 3: Add GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add the following secrets:

| Secret Name | Value |
|-------------|-------|
| `AWS_ACCESS_KEY_ID` | Your AWS Access Key ID |
| `AWS_SECRET_ACCESS_KEY` | Your AWS Secret Access Key |

### Step 4: Configure Workflow (Already Created!)

The workflow file is already created at [.github/workflows/deploy.yml](.github/workflows/deploy.yml)

**Workflow triggers:**
- ✅ Push to `main` or `master` branch
- ✅ Pull request to `main` or `master` branch
- ✅ Manual trigger via GitHub UI

**What it does:**
1. Checks out your code
2. Configures AWS credentials
3. Logs into Amazon ECR
4. Builds Docker image
5. Tags image with Git commit SHA and `latest`
6. Deploy to AWS ECS/Fargate using the ECR image
6. Set up automated testing in GitHub Actions
7. Implement multi-stage Docker builds

### Step 5: Customize Configuration

Edit [.github/workflows/deploy.yml](.github/workflows/deploy.yml) if needed:

```yaml
env:
  AWS_REGION: us-east-1        # Change to your AWS region
  ECR_REPOSITORY: my-html-app  # Change to your ECR repo name
```

### Step 6: Push to GitHub and Deploy

```bash
# Initialize git (if not already done)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit with AWS ECR deployment"

# Add remote (replace with your repo URL)
git remote add origin https://github.com/yourusername/PracticalFirst-Docker.git

# Push to GitHub
git push -u origin main
```

### Step 7: Verify Deployment

1. Go to your GitHub repository
2. Click **Actions** tab
3. You should see the workflow running
4. Once complete, go to AWS ECR Console
5. Your image should be visible in the `my-html-app` repository

### Pulling Image from ECR

To use the image from ECR:

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

# Pull the image
docker pull YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-html-app:latest

# Run it
docker run -d -p 8080:80 YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-html-app:latest
```

Replace `YOUR_AWS_ACCOUNT_ID` with your actual AWS account ID.

### GitHub Actions Workflow Explained

```yaml
on:
  push:
    branches: [main, master]  # Trigger on push to main/master
  workflow_dispatch:           # Allow manual trigger
```

**Steps breakdown:**

| Step | What it does |
|------|--------------|
| `checkout` | Gets your code |
| `configure-aws-credentials` | Authenticates with AWS using secrets |
| `login-ecr` | Logs into ECR registry |
| `build-tag-push` | Builds Docker image, tags it, and pushes to ECR |

### Image Tagging Strategy

Each push creates **two tags**:
1. **Git SHA** (e.g., `abc123def`) - Specific version
2. **latest** - Always points to most recent build

Benefits:
- ✅ Rollback to specific versions
- ✅ Always have a `latest` version
- ✅ Track which Git commit produced which image

### Cost Considerations

**AWS ECR Pricing:**
- First 500 MB/month: **FREE**
- Storage: ~$0.10/GB per month
- Data transfer: First 1 GB/month FREE

This simple app (~10 MB) will stay in the free tier!

### Troubleshooting AWS ECR

#### Authentication Failed
```bash
# Verify AWS credentials
aws sts get-caller-identity
```

#### Repository Not Found
Check ECR repository name matches the workflow:
```yaml
ECR_REPOSITORY: my-html-app  # Must match ECR repo name
```

#### GitHub Actions Permission Denied
- Verify AWS IAM user has `AmazonEC2ContainerRegistryPowerUser` policy
- Check GitHub secrets are correctly set

#### Workflow Not Triggering
- Ensure you're pushing to `main` or `master` branch
- Check workflow file is in `.github/workflows/` directory
- Verify YAML syntax is correct

## NGINX serves `index.html`
5. Response travels back to browser

## 🛑 Cleanup

### Stop the Container
```bash
docker stop html-container
```

### Remove the Container
```bash
docker rm html-container
```

### Remove the Image (optional)
```bash
docker rmi my-html-app
```

### One-liner to Stop and Remove
```bash
docker stop html-container && docker rm html-container
```

## 🔧 Troubleshooting

### Port Already in Use
If port 8080 is busy, use a different port:
```bash
docker run -d -p 8081:80 --name html-container my-html-app
```
Then visit `http://localhost:8081`

### Container Name Conflict
If the name is taken, remove the old container:
```bash
docker rm -f html-container
```

### Can't Access localhost:8080
1. Verify container is running: `docker ps`
2. Check logs: `docker logs html-container`
3. Restart container: `docker restart html-container`

## 📚 Useful Docker Commands

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker images` | List all images |
| `docker logs <container>` | View container logs |
| `docker exec -it <container> sh` | Enter container shell |
| `docker stop <container>` | Stop a container |
| `docker start <container>` | Start a stopped container |
| `docker restart <container>` | Restart a container |
| `docker rm <container>` | Remove a container |
| `docker rmi <image>` | Remove an image |

## 🎓 Next Steps

Now that you've mastered the basics, try:
1. Adding CSS styling to your HTML
2. Creating a multi-page website
3. Using environment variables
4. Creating a docker-compose.yml file
5. Pushing your image to Docker Hub

## 📝 License

This is an educational project - feel free to use and modify!

---

**🎉 Congratulations!** You've successfully dockerized your first web application!
