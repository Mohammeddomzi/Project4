# Quick Reference Guide

## GitHub Secrets Required

Add these secrets to your GitHub repository (Settings → Secrets and variables → Actions):

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `AWS_ACCESS_KEY_ID` | AWS access key for GitHub Actions user | AWS IAM Console → github-action-user → Security Credentials |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key for GitHub Actions user | AWS IAM Console → github-action-user → Security Credentials |
| `EKS_CLUSTER_NAME` | Name of your EKS cluster | `terraform output cluster_name` |
| `BACKEND_URL` | Backend service URL | `kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'` |

## Common Commands

### Terraform
```bash
# Initialize
cd setup/terraform && terraform init

# Apply infrastructure
terraform apply

# View outputs
terraform output

# Destroy infrastructure
terraform destroy
```

### Git Workflow
```bash
# Create feature branch
git checkout -b feature/your-feature

# Make changes and commit
git add .
git commit -m "Description of changes"

# Push and create PR
git push -u origin feature/your-feature

# After PR is merged, update main
git checkout main
git pull origin main
```

### Kubernetes
```bash
# Update kubeconfig
aws eks update-kubeconfig --name <cluster-name> --region us-east-1

# View pods
kubectl get pods

# View services
kubectl get services

# View logs
kubectl logs <pod-name>

# Get frontend URL
kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Get backend URL
kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Describe pod
kubectl describe pod <pod-name>

# Delete pod (will auto-restart)
kubectl delete pod <pod-name>
```

### Local Development

#### Frontend
```bash
cd starter/frontend
npm ci
npm start                                    # Development server
npm test                                     # Run tests
npm run lint                                 # Run linter
npm run build                                # Build for production
```

#### Backend
```bash
cd starter/backend
pipenv install                               # Install dependencies
pipenv run serve                             # Start dev server
pipenv run test                              # Run tests
pipenv run lint                              # Run linter
```

### Docker Commands
```bash
# Build frontend
cd starter/frontend
docker build --build-arg=REACT_APP_MOVIE_API_URL=http://localhost:5000 --tag=mp-frontend:latest .

# Run frontend
docker run -p 3000:3000 --name mp-frontend -d mp-frontend:latest

# Build backend
cd starter/backend
docker build --tag=mp-backend:latest .

# Run backend
docker run -p 5000:5000 --name mp-backend -d mp-backend:latest

# View logs
docker logs <container-name>

# Stop container
docker stop <container-name>

# Remove container
docker rm <container-name>
```

## Workflow Triggers

### Frontend CI
- ✅ Pull requests to `main` branch
- ✅ Manual trigger via GitHub Actions UI
- ✅ Only when `starter/frontend/**` files change

### Frontend CD
- ✅ Push to `main` branch
- ✅ Manual trigger via GitHub Actions UI
- ✅ Only when `starter/frontend/**` files change

### Backend CI
- ✅ Pull requests to `main` branch
- ✅ Manual trigger via GitHub Actions UI
- ✅ Only when `starter/backend/**` files change

### Backend CD
- ✅ Push to `main` branch
- ✅ Manual trigger via GitHub Actions UI
- ✅ Only when `starter/backend/**` files change

## Pipeline Flow

### CI Pipeline
```
PR Created → Lint (parallel) + Test (parallel) → Build → Success ✅
```

### CD Pipeline
```
Push to main → Test → Build + Tag (git SHA) + Push to ECR → Deploy to K8s → Success ✅
```

## Troubleshooting Quick Fixes

### "Error: Could not authenticate with Kubernetes cluster"
```bash
# Re-run the init script
cd setup && ./init.sh
```

### "Error: unauthorized to ECR repository"
```bash
# Login to ECR manually
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

### "Pipeline stuck on deployment"
```bash
# Check if pods are running
kubectl get pods

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check pod logs
kubectl logs <pod-name>
```

### "LoadBalancer stuck in pending"
```bash
# This is normal, wait 5-10 minutes for AWS to provision the load balancer
kubectl get services -w
```

## Useful AWS CLI Commands

```bash
# List ECR repositories
aws ecr describe-repositories

# List images in repository
aws ecr describe-images --repository-name frontend

# Get cluster info
aws eks describe-cluster --name <cluster-name>

# List IAM users
aws iam list-users
```

## File Structure
```
.
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml       # Frontend CI pipeline
│       ├── frontend-cd.yml       # Frontend CD pipeline
│       ├── backend-ci.yml        # Backend CI pipeline
│       └── backend-cd.yml        # Backend CD pipeline
├── starter/
│   ├── frontend/                 # React frontend app
│   │   ├── k8s/                  # Kubernetes manifests
│   │   ├── src/                  # Source code
│   │   ├── Dockerfile            # Container definition
│   │   └── package.json          # Dependencies
│   └── backend/                  # Flask backend API
│       ├── k8s/                  # Kubernetes manifests
│       ├── movies/               # API code
│       ├── Dockerfile            # Container definition
│       └── Pipfile               # Python dependencies
├── setup/
│   ├── terraform/                # Infrastructure as Code
│   └── init.sh                   # K8s setup script
├── README.md                     # Main documentation
├── SETUP_GUIDE.md               # Detailed setup instructions
└── QUICK_REFERENCE.md           # This file
```

## Important Notes

- 🚨 **Always run `terraform destroy` when finished** to avoid AWS charges
- ✅ Workflows are configured with path filters to prevent unnecessary runs
- ✅ Docker images are tagged with git SHA for traceability
- ✅ All workflows support manual trigger via `workflow_dispatch`
- ✅ CI pipelines run lint and test in parallel for speed
- ✅ CD pipelines only deploy after tests pass

## Next Steps After Setup

1. ✅ Push code to GitHub
2. ✅ Create a test PR to verify CI pipeline
3. ✅ Merge PR to trigger CD pipeline
4. ✅ Get frontend URL and test the application
5. ✅ Update `BACKEND_URL` secret if needed
6. ✅ Monitor pipelines in GitHub Actions tab

## Support Resources

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [AWS EKS Docs](https://docs.aws.amazon.com/eks/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Docker Docs](https://docs.docker.com/)

