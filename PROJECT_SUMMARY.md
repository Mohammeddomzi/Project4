# Movie Picture Pipeline - Project Summary

## 🎯 Project Overview

This project implements a complete CI/CD pipeline using GitHub Actions for a Movie Picture catalog web application. The pipeline automates testing, building, and deployment of both frontend and backend applications to an AWS EKS Kubernetes cluster.

## 📦 Project Structure

```
Movie-Picture-Pipeline/
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml      # Frontend Continuous Integration
│       ├── frontend-cd.yml      # Frontend Continuous Deployment
│       ├── backend-ci.yml       # Backend Continuous Integration
│       └── backend-cd.yml       # Backend Continuous Deployment
├── starter/
│   ├── frontend/                # React/TypeScript frontend
│   │   ├── src/                 # Source code
│   │   ├── k8s/                 # Kubernetes manifests
│   │   ├── Dockerfile           # Container definition
│   │   └── package.json         # Node.js dependencies
│   └── backend/                 # Flask/Python backend
│       ├── movies/              # API source code
│       ├── k8s/                 # Kubernetes manifests
│       ├── Dockerfile           # Container definition
│       └── Pipfile              # Python dependencies
├── setup/
│   ├── terraform/               # AWS infrastructure as code
│   └── init.sh                  # Kubernetes setup script
├── README.md                    # Main project documentation
├── SETUP_GUIDE.md              # Detailed setup instructions
├── QUICK_REFERENCE.md          # Quick command reference
├── PROJECT_SUMMARY.md          # This file
├── CODEOWNERS                  # Code ownership configuration
└── .gitignore                  # Git ignore patterns
```

## 🚀 CI/CD Workflows

### Frontend CI Workflow (`frontend-ci.yml`)

**Triggers:**
- Pull requests to `main` branch
- Changes in `starter/frontend/**` paths only
- Manual trigger via workflow_dispatch

**Jobs:**
1. **Lint** (parallel execution)
   - Setup Node.js 18
   - Install dependencies
   - Run ESLint

2. **Test** (parallel execution)
   - Setup Node.js 18
   - Install dependencies
   - Run test suite

3. **Build** (depends on lint & test)
   - Setup Node.js 18
   - Install dependencies
   - Build Docker image

### Frontend CD Workflow (`frontend-cd.yml`)

**Triggers:**
- Push to `main` branch
- Changes in `starter/frontend/**` paths only
- Manual trigger via workflow_dispatch

**Jobs:**
1. **Test**
   - Run test suite

2. **Build** (depends on test)
   - Configure AWS credentials
   - Login to Amazon ECR
   - Build Docker image with git SHA tag
   - Push image to ECR

3. **Deploy** (depends on build)
   - Configure AWS credentials
   - Update kubeconfig for EKS
   - Install kustomize
   - Update Kubernetes manifests with new image tag
   - Apply manifests to cluster

### Backend CI Workflow (`backend-ci.yml`)

**Triggers:**
- Pull requests to `main` branch
- Changes in `starter/backend/**` paths only
- Manual trigger via workflow_dispatch

**Jobs:**
1. **Lint** (parallel execution)
   - Setup Python 3.10
   - Install pipenv
   - Install dependencies
   - Run flake8

2. **Test** (parallel execution)
   - Setup Python 3.10
   - Install pipenv
   - Install dependencies
   - Run pytest

3. **Build** (depends on lint & test)
   - Build Docker image

### Backend CD Workflow (`backend-cd.yml`)

**Triggers:**
- Push to `main` branch
- Changes in `starter/backend/**` paths only
- Manual trigger via workflow_dispatch

**Jobs:**
1. **Test**
   - Run pytest test suite

2. **Build** (depends on test)
   - Configure AWS credentials
   - Login to Amazon ECR
   - Build Docker image with git SHA tag
   - Push image to ECR

3. **Deploy** (depends on build)
   - Configure AWS credentials
   - Update kubeconfig for EKS
   - Install kustomize
   - Update Kubernetes manifests with new image tag
   - Apply manifests to cluster

## 🔧 Key Features

### Workflow Features
- ✅ **Path Filtering**: Workflows trigger only when relevant code changes
- ✅ **Parallel Execution**: Lint and test jobs run simultaneously for faster feedback
- ✅ **Manual Triggers**: All workflows support manual execution
- ✅ **Dependency Management**: Build jobs only run after tests pass
- ✅ **Git SHA Tagging**: Docker images tagged with commit SHA for traceability
- ✅ **Error Handling**: Workflows fail fast on errors
- ✅ **Caching**: Node.js dependencies cached for faster builds

### Security Features
- ✅ **AWS Credentials**: Stored as GitHub Secrets
- ✅ **ECR Authentication**: Automatic login to private registries
- ✅ **IAM Roles**: Dedicated GitHub Actions user with minimal permissions
- ✅ **Kubernetes RBAC**: Proper authentication for cluster access

### Deployment Features
- ✅ **Container Orchestration**: Kubernetes for high availability
- ✅ **LoadBalancer Services**: External access to applications
- ✅ **Kustomize**: Dynamic manifest generation
- ✅ **Rolling Updates**: Zero-downtime deployments
- ✅ **AWS EKS**: Managed Kubernetes cluster

## 📋 Project Requirements Met

### ✅ CI Pipeline Requirements
- [x] Runs on pull requests to main branch
- [x] Triggered only when relevant application code changes
- [x] Can be manually triggered
- [x] Runs lint and test jobs in parallel
- [x] Runs build job only after lint and test succeed
- [x] Separate workflows for frontend and backend

### ✅ CD Pipeline Requirements
- [x] Runs on push to main branch
- [x] Triggered only when relevant application code changes
- [x] Can be manually triggered
- [x] Runs tests before building
- [x] Tags Docker images with git SHA
- [x] Pushes images to Amazon ECR
- [x] Deploys to Kubernetes cluster using kustomize
- [x] Separate workflows for frontend and backend

### ✅ Additional Requirements
- [x] Error handling in workflows
- [x] Clear workflow status visibility
- [x] Documentation for setup and usage
- [x] Infrastructure as code with Terraform
- [x] GitHub secrets configuration guide

## 🏗️ Infrastructure Components

### AWS Resources (via Terraform)
- **EKS Cluster**: Managed Kubernetes cluster
- **ECR Repositories**: Docker image storage (frontend & backend)
- **IAM User**: `github-action-user` for CI/CD access
- **VPC**: Networking infrastructure
- **Security Groups**: Network access control
- **IAM Roles**: EKS cluster and node roles

### Kubernetes Resources
- **Deployments**: Frontend and backend application deployments
- **Services**: LoadBalancer services for external access
- **Namespaces**: Default namespace for applications
- **ConfigMaps**: Application configuration (via kustomize)

## 📊 Workflow Visualization

```
┌─────────────────────────────────────────┐
│         GitHub Repository               │
│    (Frontend + Backend Code)            │
└──────────────┬──────────────────────────┘
               │
        ┌──────┴──────┐
        │             │
    ┌───▼───┐    ┌───▼────┐
    │  PR   │    │ Push   │
    │Created│    │to Main │
    └───┬───┘    └───┬────┘
        │            │
    ┌───▼─────┐  ┌──▼──────┐
    │   CI    │  │   CD    │
    │Pipeline │  │Pipeline │
    └───┬─────┘  └──┬──────┘
        │            │
    ┌───▼─────┐  ┌──▼──────┐
    │ Lint ║  │  │  Test   │
    │ Test ║  │  │  Build  │
    │ Build│  │  │  Push   │
    └───┬────┘  │  Deploy  │
        │       └──┬───────┘
     Success      │
        │       ┌─▼─────────┐
        │       │   ECR     │
        │       │ (Images)  │
        │       └─┬─────────┘
        │         │
        │       ┌─▼─────────┐
        │       │    EKS    │
        │       │ (Cluster) │
        │       └─┬─────────┘
        │         │
        │       ┌─▼─────────┐
        │       │   Apps    │
        │       │  Running  │
        │       └───────────┘
        │
    Ready for
     Merge
```

## 🎓 Technologies Used

### Frontend
- **React**: UI framework
- **TypeScript/JavaScript**: Programming language
- **Axios**: HTTP client for API calls
- **React Testing Library**: Testing framework
- **ESLint**: Code linting
- **Node.js**: Runtime environment

### Backend
- **Flask**: Python web framework
- **Python 3.10**: Programming language
- **pytest**: Testing framework
- **flake8**: Code linting
- **pipenv**: Dependency management
- **uwsgi**: WSGI server

### DevOps & Infrastructure
- **GitHub Actions**: CI/CD platform
- **Docker**: Containerization
- **Kubernetes**: Container orchestration
- **AWS EKS**: Managed Kubernetes
- **AWS ECR**: Container registry
- **Terraform**: Infrastructure as Code
- **kustomize**: Kubernetes configuration management
- **kubectl**: Kubernetes CLI

## 📝 Next Steps After Setup

1. **Create GitHub Repository**
   ```bash
   gh repo create movie-picture-pipeline --source=. --public --push
   ```

2. **Apply Terraform Infrastructure**
   ```bash
   cd setup/terraform
   terraform init
   terraform apply
   ```

3. **Configure GitHub Secrets**
   - AWS_ACCESS_KEY_ID
   - AWS_SECRET_ACCESS_KEY
   - EKS_CLUSTER_NAME
   - BACKEND_URL

4. **Initialize Kubernetes**
   ```bash
   cd setup
   ./init.sh
   ```

5. **Test CI Pipeline**
   - Create a feature branch
   - Make a change
   - Create a pull request
   - Verify workflows run

6. **Test CD Pipeline**
   - Merge pull request
   - Verify deployment to EKS
   - Access application via LoadBalancer URL

## 🔍 Monitoring & Verification

### Check Pipeline Status
- Go to GitHub repository → Actions tab
- View workflow runs, logs, and results

### Check Kubernetes Deployment
```bash
# Update kubeconfig
aws eks update-kubeconfig --name <cluster-name> --region us-east-1

# Check pods
kubectl get pods

# Check services
kubectl get services

# View logs
kubectl logs <pod-name>
```

### Access Applications
```bash
# Get frontend URL
kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Get backend URL
kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## 🎯 Success Criteria

The project is successful when:

1. ✅ All four workflow files are created and configured correctly
2. ✅ CI pipelines run on pull requests
3. ✅ CD pipelines run on push to main
4. ✅ Docker images are built and tagged with git SHA
5. ✅ Images are pushed to ECR successfully
6. ✅ Applications deploy to Kubernetes cluster
7. ✅ Frontend displays movie catalog from backend API
8. ✅ LoadBalancer URLs are accessible
9. ✅ Workflows show green checkmarks in GitHub Actions

## 🚨 Important Notes

- **Always destroy infrastructure when done**: Run `terraform destroy` to avoid AWS charges
- **Keep secrets secure**: Never commit AWS credentials to repository
- **Monitor costs**: Check AWS billing dashboard regularly
- **Update documentation**: Keep README updated with any changes
- **Test locally first**: Verify builds work locally before pushing

## 📚 Documentation Files

- **README.md**: Main project documentation
- **SETUP_GUIDE.md**: Detailed step-by-step setup instructions
- **QUICK_REFERENCE.md**: Quick command reference guide
- **PROJECT_SUMMARY.md**: This comprehensive project overview
- **setup/terraform/README.md**: Terraform-specific documentation

## 🤝 Contributing

1. Create a feature branch
2. Make changes
3. Create pull request
4. Wait for CI checks to pass
5. Get review and merge
6. CD pipeline automatically deploys

## 📧 Contact

**Developer**: imohammedomz@gmail.com

## 📄 License

See LICENSE.md for details.

---

**Project Status**: ✅ Ready for deployment

**Last Updated**: October 6, 2025

