# 🚀 Getting Started - Quick Setup Guide

## ✅ What's Already Done

- ✅ Project files copied and organized
- ✅ GitHub Actions workflows created (4 files)
- ✅ Documentation created (7 files)
- ✅ Git repository initialized
- ✅ GitHub CLI installed
- ✅ Terraform installed

## 🔄 Important: Restart Your Terminal

**You MUST restart your PowerShell/terminal now** so it can recognize the newly installed tools (`gh` and `terraform`).

1. Close your current PowerShell window
2. Open a new PowerShell window
3. Navigate back to the project:
   ```powershell
   cd "C:\Users\Administrator\Desktop\React Projects\Project4"
   ```

---

## 📋 Step-by-Step Setup (After Restarting Terminal)

### Step 1: Verify Tools are Installed

```powershell
# Check GitHub CLI
gh --version

# Check Terraform
terraform --version

# Check AWS CLI (should already be configured)
aws --version
```

All commands should show version numbers without errors.

---

### Step 2: Create GitHub Repository

```powershell
# Navigate to project directory (if not already there)
cd "C:\Users\Administrator\Desktop\React Projects\Project4"

# Authenticate with GitHub
gh auth login
# Follow the prompts:
#   - Select: GitHub.com
#   - Select: HTTPS
#   - Press: Y or Enter
#   - Select: Login with a web browser
#   - Copy the code and press Enter
#   - Login and authorize in browser

# Create repository and push
gh repo create movie-picture-pipeline --source=. --public --push
```

**Alternative Manual Method:**
1. Go to https://github.com → New repository
2. Name: `movie-picture-pipeline`
3. Make it **Public**
4. Don't initialize with anything
5. Create repository
6. Then run:
   ```powershell
   git remote add origin https://github.com/YOUR_USERNAME/movie-picture-pipeline.git
   git push -u origin main
   ```

---

### Step 3: Verify GitHub Repository

1. Go to your repository on GitHub
2. Check that all files are there
3. Go to **Actions** tab - you should see the workflows

---

### Step 4: Setup AWS Infrastructure with Terraform

```powershell
# Navigate to terraform directory
cd "C:\Users\Administrator\Desktop\React Projects\Project4\setup\terraform"

# Initialize Terraform
terraform init

# Review what will be created
terraform plan

# Apply the configuration (creates AWS resources)
terraform apply
# Type 'yes' when prompted
# ⏱️ This takes 10-15 minutes

# After completion, save the outputs
terraform output
```

**Important**: Save these Terraform outputs somewhere safe:
- `frontend_ecr` - ECR repository URL for frontend
- `backend_ecr` - ECR repository URL for backend  
- `cluster_name` - EKS cluster name
- `github_action_user_arn` - IAM user ARN

---

### Step 5: Create AWS Access Keys for GitHub Actions

1. Go to **AWS Console** → **IAM** → **Users**
2. Click on **github-action-user**
3. Go to **Security Credentials** tab
4. Click **Create access key**
5. Select **Application running outside AWS** → Next → Create
6. **Save both keys** (you won't see the secret again):
   - Access Key ID: `__________________________`
   - Secret Access Key: `__________________________`

---

### Step 6: Configure GitHub Secrets

Go to your GitHub repository:
1. Click **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add these 4 secrets one by one:

| Secret Name | Value |
|-------------|-------|
| `AWS_ACCESS_KEY_ID` | From Step 5 |
| `AWS_SECRET_ACCESS_KEY` | From Step 5 |
| `EKS_CLUSTER_NAME` | From Step 4 (terraform output) |
| `BACKEND_URL` | `backend.default.svc.cluster.local` (temporary) |

---

### Step 7: Initialize Kubernetes Cluster

```powershell
# Navigate to setup directory
cd "C:\Users\Administrator\Desktop\React Projects\Project4\setup"

# Run the init script (use bash if on Windows with Git Bash)
bash init.sh

# OR if you have WSL:
wsl bash init.sh
```

If you don't have bash, you can run the commands manually:
```powershell
# Get cluster name from terraform output
$CLUSTER_NAME = "YOUR_CLUSTER_NAME_HERE"

# Update kubeconfig
aws eks update-kubeconfig --name $CLUSTER_NAME --region us-east-1

# Verify connection
kubectl get nodes
```

---

### Step 8: Test Your CI/CD Pipeline

```powershell
# Create a test branch
git checkout -b test-ci

# Make a small change
echo "# Test" >> README.md

# Commit and push
git add .
git commit -m "Test: CI pipeline"
git push -u origin test-ci
```

Then:
1. Go to GitHub → Your repository
2. Click **Compare & pull request**
3. Create the pull request
4. Go to **Actions** tab - workflows should be running!
5. Wait for them to complete (should pass ✅)

---

### Step 9: Trigger CD Pipeline

1. Merge the pull request on GitHub
2. Go to **Actions** tab
3. CD workflows should start automatically
4. Wait for completion (5-10 minutes)

---

### Step 10: Verify Deployment

```powershell
# Check if pods are running
kubectl get pods

# Check services
kubectl get services

# Get frontend URL
kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Get backend URL (update GitHub secret with this)
kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Wait 5-10 minutes for LoadBalancers to provision, then:
- Open frontend URL in browser
- You should see the Movie List application!

---

## 📚 Additional Resources

- **Detailed Guide**: `DEPLOYMENT_CHECKLIST.md` - Complete checklist with troubleshooting
- **Setup Instructions**: `SETUP_GUIDE.md` - In-depth setup guide
- **Quick Commands**: `QUICK_REFERENCE.md` - Common commands reference
- **Architecture**: `PROJECT_SUMMARY.md` - Full project overview

---

## 🚨 Common Issues

### Issue: Commands not recognized after installation
**Solution**: Restart your PowerShell/Terminal

### Issue: `gh auth login` fails
**Solution**: Make sure you're using a web browser login option

### Issue: Terraform apply fails
**Solution**: Verify AWS CLI is configured with `aws sts get-caller-identity`

### Issue: kubectl can't connect
**Solution**: Run `aws eks update-kubeconfig --name <cluster-name> --region us-east-1`

### Issue: LoadBalancer stuck in pending
**Solution**: Wait 10-15 minutes - AWS needs time to provision

---

## ⚠️ Important Reminders

1. **GitHub repository MUST be public** for free GitHub Actions
2. **Save Terraform outputs** - you'll need them for GitHub Secrets
3. **Save AWS credentials** - you can't retrieve the secret key later
4. **Destroy infrastructure when done** - Run `terraform destroy` to avoid charges
5. **Check AWS billing** - Monitor your AWS costs

---

## 🎯 Success Criteria

Your setup is complete when:
- ✅ Repository is on GitHub
- ✅ All 4 workflows show in Actions tab
- ✅ Terraform has created AWS resources
- ✅ GitHub Secrets are configured
- ✅ kubectl can connect to EKS cluster
- ✅ CI pipeline runs on pull requests
- ✅ CD pipeline deploys to Kubernetes
- ✅ Frontend shows movie list in browser

---

## 📞 Need Help?

Follow the detailed checklist in `DEPLOYMENT_CHECKLIST.md` - it has every command you need with checkboxes!

---

**Good luck! 🚀 Your Movie Picture Pipeline is ready to deploy!**

---

**Email**: imohammedomz@gmail.com  
**Date**: October 6, 2025

