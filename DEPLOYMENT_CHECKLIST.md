# Movie Picture Pipeline - Deployment Checklist

## 📋 Pre-Deployment Checklist

Use this checklist to ensure you complete all steps in the correct order.

---

## Phase 1: GitHub Repository Setup

- [ ] **1.1** Verify Git is initialized
  ```bash
  cd "C:\Users\Administrator\Desktop\React Projects\Project4"
  git status
  ```

- [ ] **1.2** Create GitHub repository
  ```bash
  gh auth login  # If not already logged in
  gh repo create movie-picture-pipeline --source=. --public --push
  ```
  
  Or manually:
  - Go to github.com
  - Create new repository named `movie-picture-pipeline`
  - Make it public (required for free GitHub Actions)
  - Don't initialize with README (we already have files)
  - Copy the repository URL

- [ ] **1.3** Push code to GitHub
  ```bash
  git remote add origin https://github.com/YOUR_USERNAME/movie-picture-pipeline.git
  git push -u origin main
  ```

- [ ] **1.4** Verify files are on GitHub
  - Go to your repository URL
  - Check that all files are visible
  - Verify `.github/workflows` directory contains 4 workflow files

---

## Phase 2: AWS Infrastructure Setup

- [ ] **2.1** Verify AWS CLI is configured
  ```bash
  aws sts get-caller-identity
  ```
  Should return your AWS account details.

- [ ] **2.2** Navigate to Terraform directory
  ```bash
  cd "C:\Users\Administrator\Desktop\React Projects\Project4\setup\terraform"
  ```

- [ ] **2.3** Initialize Terraform
  ```bash
  terraform init
  ```

- [ ] **2.4** Review planned changes
  ```bash
  terraform plan
  ```
  Review what will be created (EKS cluster, ECR repos, IAM user, etc.)

- [ ] **2.5** Apply Terraform configuration
  ```bash
  terraform apply
  ```
  Type `yes` when prompted.
  ⏱️ This will take 10-15 minutes.

- [ ] **2.6** Save Terraform outputs
  ```bash
  terraform output
  ```
  Copy and save these values:
  - `frontend_ecr`: ___________________________________
  - `backend_ecr`: ___________________________________
  - `cluster_name`: ___________________________________
  - `github_action_user_arn`: ___________________________________

---

## Phase 3: AWS Credentials for GitHub Actions

- [ ] **3.1** Go to AWS Console
  Navigate to: IAM → Users

- [ ] **3.2** Open github-action-user
  Click on the user created by Terraform

- [ ] **3.3** Go to Security Credentials tab

- [ ] **3.4** Create Access Key
  - Click "Create access key"
  - Select "Application running outside AWS"
  - Click "Next" → "Create access key"

- [ ] **3.5** Save credentials (IMPORTANT!)
  - Access Key ID: ___________________________________
  - Secret Access Key: ___________________________________
  
  ⚠️ You won't be able to see the secret again!

---

## Phase 4: Configure Kubernetes

- [ ] **4.1** Navigate to setup directory
  ```bash
  cd "C:\Users\Administrator\Desktop\React Projects\Project4\setup"
  ```

- [ ] **4.2** Make init script executable (if on Linux/Mac)
  ```bash
  chmod +x init.sh
  ```

- [ ] **4.3** Run initialization script
  ```bash
  ./init.sh
  ```
  Or on Windows:
  ```bash
  bash init.sh
  ```

- [ ] **4.4** Verify kubectl access
  ```bash
  aws eks update-kubeconfig --name <cluster_name> --region us-east-1
  kubectl get nodes
  ```
  Should show EKS nodes.

---

## Phase 5: Configure GitHub Secrets

- [ ] **5.1** Go to GitHub repository Settings

- [ ] **5.2** Navigate to Secrets
  Settings → Secrets and variables → Actions → New repository secret

- [ ] **5.3** Add AWS_ACCESS_KEY_ID
  - Name: `AWS_ACCESS_KEY_ID`
  - Value: (from Phase 3.5)

- [ ] **5.4** Add AWS_SECRET_ACCESS_KEY
  - Name: `AWS_SECRET_ACCESS_KEY`
  - Value: (from Phase 3.5)

- [ ] **5.5** Add EKS_CLUSTER_NAME
  - Name: `EKS_CLUSTER_NAME`
  - Value: (from Phase 2.6)

- [ ] **5.6** Add BACKEND_URL (temporary placeholder)
  - Name: `BACKEND_URL`
  - Value: `backend.default.svc.cluster.local`
  
  ⚠️ You'll update this after first deployment

---

## Phase 6: Test CI Pipeline

- [ ] **6.1** Create test branch
  ```bash
  cd "C:\Users\Administrator\Desktop\React Projects\Project4"
  git checkout -b test-ci
  ```

- [ ] **6.2** Make a small change
  ```bash
  # Edit starter/frontend/README.md or add a comment to a file
  echo "# Test CI" >> starter/frontend/README.md
  ```

- [ ] **6.3** Commit and push
  ```bash
  git add .
  git commit -m "Test: Trigger CI pipeline"
  git push -u origin test-ci
  ```

- [ ] **6.4** Create Pull Request on GitHub
  - Go to your repository
  - Click "Compare & pull request"
  - Create the PR

- [ ] **6.5** Verify CI workflows run
  - Go to Actions tab
  - You should see workflows running
  - Wait for them to complete
  - All checks should pass ✅

---

## Phase 7: Test CD Pipeline

- [ ] **7.1** Merge Pull Request
  - Click "Merge pull request" on GitHub
  - Confirm merge

- [ ] **7.2** Verify CD workflows run
  - Go to Actions tab
  - Frontend CD and/or Backend CD should be running
  - Wait for completion (5-10 minutes)

- [ ] **7.3** Check deployment status
  ```bash
  kubectl get pods
  ```
  Should show frontend and/or backend pods running.

- [ ] **7.4** Check services
  ```bash
  kubectl get services
  ```
  Should show LoadBalancer services.

---

## Phase 8: Get Backend URL

- [ ] **8.1** Get backend LoadBalancer URL
  ```bash
  kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
  ```
  URL: ___________________________________

- [ ] **8.2** Wait for LoadBalancer to be ready
  ⏱️ Can take 5-10 minutes for first deployment

- [ ] **8.3** Test backend endpoint
  ```bash
  curl http://<backend-url>/movies
  ```
  Should return JSON with movie data.

- [ ] **8.4** Update GitHub Secret
  - Go to GitHub → Settings → Secrets
  - Edit `BACKEND_URL` secret
  - Set value to: `http://<backend-loadbalancer-url>`

---

## Phase 9: Deploy Frontend with Correct Backend URL

- [ ] **9.1** Make a small change to frontend
  ```bash
  git checkout main
  git pull
  git checkout -b update-backend-url
  echo "# Updated" >> starter/frontend/README.md
  git add .
  git commit -m "Update: Trigger frontend redeploy with backend URL"
  git push -u origin update-backend-url
  ```

- [ ] **9.2** Create and merge PR

- [ ] **9.3** Wait for CD to complete

---

## Phase 10: Verify Complete Deployment

- [ ] **10.1** Get frontend URL
  ```bash
  kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
  ```
  URL: ___________________________________

- [ ] **10.2** Wait for LoadBalancer
  ⏱️ Can take 5-10 minutes

- [ ] **10.3** Open frontend in browser
  Navigate to: `http://<frontend-url>`

- [ ] **10.4** Verify movie list displays
  - Should see "Movie List" heading
  - Should see movies from backend:
    - Top Gun: Maverick
    - Sonic the Hedgehog
    - A Quiet Place

- [ ] **10.5** Take screenshot for documentation
  Save screenshot showing working application.

---

## Phase 11: Verify All Workflows

- [ ] **11.1** Check all workflows ran successfully
  - GitHub → Actions tab
  - All 4 workflows should show green checkmarks:
    - ✅ Frontend CI
    - ✅ Frontend CD
    - ✅ Backend CI
    - ✅ Backend CD

- [ ] **11.2** Verify ECR images
  ```bash
  aws ecr describe-images --repository-name frontend --region us-east-1
  aws ecr describe-images --repository-name backend --region us-east-1
  ```
  Should show images tagged with git SHA.

- [ ] **11.3** Verify Kubernetes resources
  ```bash
  kubectl get all
  ```
  Should show deployments, pods, and services running.

---

## Phase 12: Documentation & Cleanup

- [ ] **12.1** Update README badges
  - Edit README.md
  - Replace `YOUR_USERNAME` with your actual GitHub username

- [ ] **12.2** Commit and push updates
  ```bash
  git add README.md
  git commit -m "Update README badges"
  git push
  ```

- [ ] **12.3** Document your setup
  - Save all URLs and credentials securely
  - Take screenshots of successful deployments
  - Note any customizations or issues encountered

- [ ] **12.4** (OPTIONAL) Destroy infrastructure when done
  ```bash
  cd setup/terraform
  terraform destroy
  ```
  Type `yes` to confirm.
  
  ⚠️ Only do this when completely finished!

---

## ✅ Success Criteria

Your project is successfully deployed when:

- ✅ All 4 GitHub Actions workflows exist
- ✅ CI workflows run on pull requests
- ✅ CD workflows run on push to main
- ✅ Docker images in ECR tagged with git SHA
- ✅ Kubernetes pods running (2/2 ready)
- ✅ LoadBalancer services accessible
- ✅ Frontend displays movie list
- ✅ Backend API returns movie data
- ✅ All workflows show green status

---

## 🚨 Troubleshooting

If something doesn't work, check:

1. **GitHub Actions failing?**
   - Check workflow logs in Actions tab
   - Verify GitHub secrets are set correctly
   - Check IAM permissions in AWS

2. **Terraform apply fails?**
   - Verify AWS credentials are configured
   - Check you have sufficient AWS permissions
   - Review error messages in Terraform output

3. **Kubernetes deployment fails?**
   - Verify init.sh ran successfully
   - Check kubectl can connect to cluster
   - Review pod logs: `kubectl logs <pod-name>`

4. **LoadBalancer not accessible?**
   - Wait 10-15 minutes for provisioning
   - Check security groups in AWS Console
   - Verify service type is LoadBalancer

5. **Frontend shows no movies?**
   - Verify BACKEND_URL secret is correct
   - Check backend is running: `kubectl get pods`
   - Test backend directly: `curl http://<backend-url>/movies`

---

## 📞 Need Help?

Refer to these documents:
- `SETUP_GUIDE.md` - Detailed setup instructions
- `QUICK_REFERENCE.md` - Common commands
- `PROJECT_SUMMARY.md` - Architecture overview
- `README.md` - Main documentation

---

**Good luck with your deployment! 🚀**

---

**Checklist Version**: 1.0  
**Last Updated**: October 6, 2025  
**Project**: Movie Picture Pipeline

