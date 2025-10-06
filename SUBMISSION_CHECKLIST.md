# Movie Picture Pipeline - Submission Checklist

## 📋 **Rubric Requirements - Complete Checklist**

---

## ✅ **COMPLETED Items:**

### **1. Frontend CI Workflow** ✅

- [x] File: `.github/workflows/frontend-ci.yml`
- [x] Name: "Frontend Continuous Integration"
- [x] **Lint Job** with all required steps:
  - [x] Checkout code
  - [x] Setup NodeJS
  - [x] Cache action (npm cache)
  - [x] Install dependencies (npm ci)
  - [x] Run npm run lint
- [x] **Test Job** with all required steps:
  - [x] Checkout code
  - [x] Setup NodeJS
  - [x] Cache action (npm cache)
  - [x] Install dependencies (npm ci)
  - [x] Run npm test (CI=true npm test)
- [x] **Lint & Test run in parallel** ✅
- [x] **Build Job** (needs: [lint, test]):
  - [x] Checkout code
  - [x] Docker build step
- [x] Triggers on pull_request ✅
- [x] Manual trigger (workflow_dispatch) ✅
- [x] All jobs pass ✅
- [x] **Screenshot**: `1-frontend-ci-workflow-success.png`

---

### **2. Backend CI Workflow** ✅

- [x] File: `.github/workflows/backend-ci.yml`
- [x] Name: "Backend Continuous Integration"
- [x] **Lint Job** (pipenv run lint)
- [x] **Test Job** (pipenv run test)
- [x] **Lint & Test run in parallel** ✅
- [x] **Build Job** (needs: [lint, test])
- [x] Docker build step
- [x] Triggers on pull_request ✅
- [x] Manual trigger (workflow_dispatch) ✅
- [x] All jobs pass ✅
- [x] **Screenshot**: `5-backend-ci-workflow-success.png`

---

### **3. Frontend CD Workflow** ✅

- [x] File: `.github/workflows/frontend-cd.yml`
- [x] Name: "Frontend Continuous Deployment"
- [x] **Test Job** (runs tests)
- [x] **Build Job** (needs: test):
  - [x] Uses aws-actions/amazon-ecr-login ✅
  - [x] Accesses GitHub Secrets (no hardcoded credentials) ✅
  - [x] Docker build with --build-arg REACT_APP_MOVIE_API_URL ✅
  - [x] Tagged with git SHA ✅
  - [x] Pushes to ECR ✅
- [x] **Deploy Job** (needs: build):
  - [x] Uses kubectl to deploy
  - [x] Uses kustomize
  - [x] Updates image with git SHA tag
- [x] Triggers on push to main ✅
- [x] Manual trigger (workflow_dispatch) ✅
- [x] All jobs pass ✅
- [x] **Screenshots**:
  - [x] `2-frontend-cd-workflow-success.png`
  - [x] `7-ecr-frontend-image-pushed.png`

---

### **4. Backend CD Workflow** ✅

- [x] File: `.github/workflows/backend-cd.yml`
- [x] Name: "Backend Continuous Deployment"
- [x] **Test Job** (runs tests)
- [x] **Build Job** (needs: test):
  - [x] Uses aws-actions/amazon-ecr-login ✅
  - [x] Accesses GitHub Secrets (no hardcoded credentials) ✅
  - [x] Docker build
  - [x] Tagged with git SHA ✅
  - [x] Pushes to ECR ✅
- [x] **Deploy Job** (needs: build):
  - [x] Uses kubectl to deploy
  - [x] Uses kustomize
  - [x] Updates image with git SHA tag
- [x] Triggers on push to main ✅
- [x] Manual trigger (workflow_dispatch) ✅
- [x] All jobs pass ✅
- [x] **Screenshots**:
  - [x] `4-backend-cd-workflow-success.png`
  - [x] `6-ecr-backend-image-pushed.png`

---

### **5. Kubernetes Deployment** ✅

- [x] Both applications deployed to EKS cluster
- [x] Pods running successfully
- [x] Services created (LoadBalancer)
- [x] **Screenshot**: `3-kubernetes-pods-and-services-deployed.png`

---

## ⚠️ **CRITICAL: MISSING Screenshots for Submission**

### **Required by Rubric:**

> **Frontend CD**: "Submit a working URL or screenshots showing the frontend application is functioning. The frontend should be able to pull the list of movies and verify the environment variable was passed correctly"

> **Backend CD**: "Submit a working URL or screenshot showing that the Backend API returns the list of movies"

---

## 📸 **YOU MUST ADD These 2 Screenshots:**

### **Screenshot 8: Backend API Response** ⚠️ REQUIRED

**Action**: Open browser or use curl and take screenshot showing:

- **URL**: `http://a66d8d8f200a24c31b1a64e62f705ccc-910932871.us-east-1.elb.amazonaws.com/movies`
- **Expected Output**:

```json
{
  "movies": [
    { "id": "123", "title": "Top Gun: Maverick" },
    { "id": "456", "title": "Sonic the Hedgehog" },
    { "id": "789", "title": "A Quiet Place" },
    { "id": "101", "title": "The Matrix" }
  ]
}
```

- **Save as**: `8-backend-api-movies-response.png`

---

### **Screenshot 9: Frontend Application Working** ⚠️ REQUIRED

**Action**: Open browser and take screenshot showing:

- **URL**: `http://aad7d89105ccb4addac5065d7107d19c-1405076523.us-east-1.elb.amazonaws.com`
- **Must show**:
  - "Movie List" heading
  - List of movies from backend API
  - All 4 movies displayed correctly
- **Save as**: `9-frontend-application-working.png`

**Note**: If frontend doesn't show movies, you need to:

1. Update GitHub Secret `BACKEND_URL` to: `a66d8d8f200a24c31b1a64e62f705ccc-910932871.us-east-1.elb.amazonaws.com`
2. Redeploy frontend (create PR and merge)
3. Wait for deployment to complete
4. Then take screenshot

---

## 📋 **Complete Screenshot List (9 total):**

1. ✅ `1-frontend-ci-workflow-success.png` - Frontend CI (lint, test, build)
2. ✅ `2-frontend-cd-workflow-success.png` - Frontend CD deployment
3. ✅ `3-kubernetes-pods-and-services-deployed.png` - kubectl output
4. ✅ `4-backend-cd-workflow-success.png` - Backend CD deployment
5. ✅ `5-backend-ci-workflow-success.png` - Backend CI (lint, test, build)
6. ✅ `6-ecr-backend-image-pushed.png` - Backend image in ECR
7. ✅ `7-ecr-frontend-image-pushed.png` - Frontend image in ECR
8. ⚠️ `8-backend-api-movies-response.png` - **MISSING - REQUIRED**
9. ⚠️ `9-frontend-application-working.png` - **MISSING - REQUIRED**

---

## ✅ **Security Checklist:**

- [x] No AWS credentials in workflow files (using GitHub Secrets) ✅
- [x] All workflows use secrets for AWS access ✅
- [x] No hardcoded credentials anywhere ✅

---

## ✅ **Workflow Validation:**

- [x] All workflows run without errors ✅
- [x] All tests pass ✅
- [x] Build steps complete successfully ✅
- [x] Docker images pushed to ECR ✅
- [x] Applications deployed to Kubernetes ✅
- [x] Path filters work correctly ✅
- [x] Manual triggers work ✅

---

## 🎯 **Bonus Points (Stand Out Suggestions):**

Optional improvements to consider:

- [ ] Custom reusable actions
- [ ] Dependency caching (already implemented!)
- [ ] Docker layer caching
- [ ] PR comments with workflow status
- [ ] Automated testing notifications

---

## 📦 **Final Submission Checklist:**

- [ ] All 9 screenshots in Screenshots folder
- [ ] Screenshots clearly show required information
- [ ] All workflows in .github/workflows/ directory
- [ ] README.md updated with badges
- [ ] Repository is public
- [ ] All workflows have run successfully
- [ ] Applications are accessible via LoadBalancer URLs

---

## 🔗 **URLs for Testing:**

### Frontend:

```
http://aad7d89105ccb4addac5065d7107d19c-1405076523.us-east-1.elb.amazonaws.com
```

### Backend API:

```
http://a66d8d8f200a24c31b1a64e62f705ccc-910932871.us-east-1.elb.amazonaws.com/movies
```

---

## 🎉 **Current Status: 88% Complete**

**You're almost done! Just need 2 more screenshots showing the working applications!**

---

**Last Updated**: October 6, 2025
