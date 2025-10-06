# Movie Picture Pipeline - Setup Guide

## Overview

This guide will walk you through setting up the complete CI/CD pipeline for the Movie Picture application.

## Prerequisites

- GitHub account
- AWS account with appropriate permissions
- AWS CLI configured
- Git installed locally

## Step 1: Initialize Git Repository

From your project workspace, run the following commands:

```bash
# Initialize git repository
git init

# Configure git with your email
git config user.email "imohammedomz@gmail.com"
git config user.name "Your Name"

# Stage all files
git add .

# Initial commit
git commit -m "Initial commit: Movie Picture Pipeline setup"
```

## Step 2: Create GitHub Repository

```bash
# Create a public GitHub repository (required for free GitHub Actions)
gh repo create movie-picture-pipeline --source=. --public --push

# Or manually create repository on GitHub and push
git remote add origin https://github.com/YOUR_USERNAME/movie-picture-pipeline.git
git branch -M main
git push -u origin main
```

## Step 3: Setup AWS Infrastructure with Terraform

The Terraform configuration will create:

- EKS Cluster for Kubernetes deployments
- ECR repositories for frontend and backend Docker images
- IAM user for GitHub Actions
- Necessary networking and security resources

```bash
# Navigate to terraform directory
cd setup/terraform

# Initialize Terraform
terraform init

# Review the planned changes
terraform plan

# Apply the configuration (type 'yes' when prompted)
terraform apply
```

**Important**: Save the Terraform outputs - you'll need them for GitHub Secrets:

```bash
terraform output
```

You should see outputs for:

- `frontend_ecr`: Frontend ECR repository URL
- `backend_ecr`: Backend ECR repository URL
- `cluster_name`: EKS cluster name
- `github_action_user_arn`: IAM user ARN for GitHub Actions

## Step 4: Generate AWS Access Keys for GitHub Actions

1. Go to AWS Console → IAM → Users
2. Click on `github-action-user`
3. Navigate to `Security Credentials` tab
4. Click `Create access key`
5. Select `Application running outside AWS` → Next → Create access key
6. **Save these credentials** - you'll need them for GitHub Secrets

## Step 5: Configure GitHub Actions User in Kubernetes

This step adds the GitHub Actions IAM user to the Kubernetes cluster's authentication configuration:

```bash
# Navigate to setup directory
cd setup

# Make the script executable (if on Linux/Mac)
chmod +x init.sh

# Run the initialization script
./init.sh
```

## Step 6: Configure GitHub Secrets

Go to your GitHub repository → Settings → Secrets and variables → Actions → New repository secret

Add the following secrets:

1. **AWS_ACCESS_KEY_ID**: From Step 4
2. **AWS_SECRET_ACCESS_KEY**: From Step 4
3. **EKS_CLUSTER_NAME**: From Terraform output (cluster_name)
4. **BACKEND_URL**: Will be available after first deployment, format: `backend-service-url`

## Step 7: Get Backend URL for Frontend

After deploying the backend for the first time, get the backend service URL:

```bash
# Update kubeconfig
aws eks update-kubeconfig --name <cluster_name> --region us-east-1

# Get backend service URL
kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Add this URL to GitHub Secrets as `BACKEND_URL`.

## Step 8: Testing the CI/CD Pipeline

### Test Continuous Integration (CI)

1. Create a new branch:

   ```bash
   git checkout -b test-ci
   ```

2. Make a small change to frontend or backend code

3. Commit and push:

   ```bash
   git add .
   git commit -m "Test CI pipeline"
   git push -u origin test-ci
   ```

4. Create a Pull Request on GitHub
5. Watch the CI workflow run (lint, test, build jobs should execute in parallel)

### Test Continuous Deployment (CD)

1. Merge the Pull Request to main branch
2. Watch the CD workflow run (test → build → deploy)
3. Verify deployment:

   ```bash
   # Check pods are running
   kubectl get pods

   # Check services
   kubectl get services

   # Get frontend URL
   kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
   ```

4. Open the frontend URL in your browser to see the Movie Picture application

## Workflow Files Explained

### Frontend CI (`frontend-ci.yml`)

- **Triggers**: Pull requests to main, manual dispatch
- **Jobs**:
  - Lint (parallel)
  - Test (parallel)
  - Build (runs after lint & test succeed)

### Frontend CD (`frontend-cd.yml`)

- **Triggers**: Push to main, manual dispatch
- **Jobs**:
  - Test
  - Build (tags with git SHA, pushes to ECR)
  - Deploy (applies K8s manifests with new image)

### Backend CI (`backend-ci.yml`)

- **Triggers**: Pull requests to main, manual dispatch
- **Jobs**:
  - Lint (parallel)
  - Test (parallel)
  - Build (runs after lint & test succeed)

### Backend CD (`backend-cd.yml`)

- **Triggers**: Push to main, manual dispatch
- **Jobs**:
  - Test
  - Build (tags with git SHA, pushes to ECR)
  - Deploy (applies K8s manifests with new image)

## Path Filtering

The workflows are configured to trigger only when relevant code changes:

- Frontend workflows: `starter/frontend/**`
- Backend workflows: `starter/backend/**`

This prevents unnecessary pipeline runs when only infrastructure or documentation changes.

## Troubleshooting

### Pipeline Fails at Test Stage

- Check that dependencies are correctly installed
- Verify test files haven't been modified incorrectly
- Review test output in GitHub Actions logs

### Pipeline Fails at Build Stage

- Verify Dockerfile syntax is correct
- Check Docker build arguments are properly set
- Review build logs for specific errors

### Pipeline Fails at Deploy Stage

- Verify AWS credentials are correctly set in GitHub Secrets
- Check EKS cluster is running: `aws eks describe-cluster --name <cluster_name>`
- Verify kubectl can connect: `kubectl get nodes`
- Check ECR repository exists and image was pushed successfully

### Application Not Accessible After Deployment

- Check pods are running: `kubectl get pods`
- Verify services are created: `kubectl get services`
- Check service logs: `kubectl logs <pod-name>`
- Wait a few minutes for LoadBalancer to provision (especially on first deployment)

## Cleanup / Teardown

**IMPORTANT**: Remember to destroy AWS resources when done to avoid charges:

```bash
# Navigate to terraform directory
cd setup/terraform

# Destroy all resources
terraform destroy
```

Type `yes` when prompted to confirm destruction.

## Local Development

### Frontend

```bash
cd starter/frontend
npm ci
npm start
```

### Backend

```bash
cd starter/backend
pipenv install
pipenv run serve
```

## Architecture Overview

```
┌─────────────────┐
│   GitHub Repo   │
└────────┬────────┘
         │
    ┌────┴────┐
    │  PR     │ Push to main
    │         │
┌───▼───┐ ┌──▼───┐
│  CI   │ │  CD  │
│       │ │      │
│ Lint  │ │ Test │
│ Test  │ │ Build│
│ Build │ │Deploy│
└───────┘ └──┬───┘
             │
         ┌───▼────┐
         │  ECR   │
         └───┬────┘
             │
         ┌───▼────┐
         │  EKS   │
         │  K8s   │
         └────────┘
```

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform Documentation](https://www.terraform.io/docs/)

## Support

For issues or questions, please create an issue in the GitHub repository.
