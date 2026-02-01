# Testing GitOps Workflow - Checklist

## 🎯 Goal
Verify complete GitOps workflow: commit → GitVersion → CI/CD → ECR → FluxCD → Kubernetes

---

## ✅ Pre-requisites

### 1. GitHub Secrets (Verus-elixir repo)
```bash
# Check if secrets are set:
# - AWS_ACCESS_KEY_ID
# - AWS_SECRET_ACCESS_KEY
# - FLUX_REPO_TOKEN (GitHub PAT with repo scope)
```

### 2. FluxCD Installed
```bash
# Check FluxCD is running
flux check

# Should show:
# ✔ image-reflector-controller
# ✔ image-automation-controller
# ✔ source-controller
# ✔ kustomize-controller
# ✔ helm-controller
```

### 3. AWS ECR Access
```bash
# Verify ECR repository exists
aws ecr describe-repositories --repository-names knowledge_graph_microservice --region us-east-2

# Should return repository details
```

---

## 📝 Step-by-Step Test

### Step 1: Commit Changes to Verus-elixir

```bash
cd C:\Users\dmytro\repos\verus\Verus-elixir

# Add all new files
git add GitVersion.yml .github/workflows/ci.yml helm/

# Commit with semver tag
git commit -m "feat: add GitOps deployment with Helm and GitVersion +semver: minor"

# Push to dev branch
git push origin dev
```

**Expected:**
- ✅ Commit pushed successfully
- ✅ GitHub Actions workflow triggered

---

### Step 2: Monitor GitHub Actions

```bash
# Open in browser:
# https://github.com/DistrictCyber02/Verus-elixir/actions

# Or use GitHub CLI:
gh run list --repo DistrictCyber02/Verus-elixir
gh run watch
```

**Expected workflow steps:**
1. ✅ **Test** - Run quality checks
2. ✅ **GitVersion** - Calculate version (e.g., `0.2.0-alpha.1`)
3. ✅ **Build & Push** - Build Docker image and push to ECR
   - Tags: `0.2.0-alpha.1-abc1234`, `0.2.0-alpha.1`, `latest`

**Check logs for:**
```
GitVersion outputs:
  SemVer: 0.2.0-alpha.1
  FullSemVer: 0.2.0-alpha.1
  BranchName: dev
  Sha: abc1234

Docker push:
  Pushed: 010438496771.dkr.ecr.us-east-2.amazonaws.com/knowledge_graph_microservice:0.2.0-alpha.1-abc1234
  Pushed: 010438496771.dkr.ecr.us-east-2.amazonaws.com/knowledge_graph_microservice:0.2.0-alpha.1
  Pushed: 010438496771.dkr.ecr.us-east-2.amazonaws.com/knowledge_graph_microservice:latest
```

---

### Step 3: Verify Image in ECR

```bash
# List images in ECR
aws ecr list-images \
  --repository-name knowledge_graph_microservice \
  --region us-east-2 \
  --query 'imageIds[*].imageTag' \
  --output table

# Should see:
# - 0.2.0-alpha.1-abc1234
# - 0.2.0-alpha.1
# - latest
```

**Expected:**
- ✅ New image tags visible in ECR
- ✅ Image pushed within last few minutes

---

### Step 4: Apply FluxCD Manifests

```bash
cd C:\Users\dmytro\repos\flux

# Commit flux manifests
git add apps/knowledge-graph/ docs/
git commit -m "Add knowledge-graph FluxCD automation"
git push origin main

# Apply to Kubernetes
kubectl apply -k apps/knowledge-graph/

# Expected output:
# namespace/default unchanged (or created)
# gitrepository.source.toolkit.fluxcd.io/verus-elixir created
# imagerepository.image.toolkit.fluxcd.io/knowledge-graph created
# imagepolicy.image.toolkit.fluxcd.io/knowledge-graph created
# helmrelease.helm.toolkit.fluxcd.io/knowledge-graph created
# imageupdateautomation.image.toolkit.fluxcd.io/knowledge-graph created
```

**Expected:**
- ✅ All manifests applied successfully
- ✅ No errors

---

### Step 5: Monitor FluxCD Components

```bash
# Check all FluxCD resources
flux get all

# Expected output:
# NAME                          REVISION        SUSPENDED       READY
# gitrepository/verus-elixir    main@sha1:...   False           True
# 
# NAME                          LAST SCAN       SUSPENDED       READY
# imagerepository/knowledge-graph  2m ago       False           True
# 
# NAME                          LATEST IMAGE                    READY
# imagepolicy/knowledge-graph   0.2.0-alpha.1-abc1234          True
# 
# NAME                          REVISION        SUSPENDED       READY
# helmrelease/knowledge-graph   0.2.0           False           True
```

**Check each component:**

#### ImageRepository
```bash
flux get images repository knowledge-graph

# Should show:
# NAME              LAST SCAN  SUSPENDED  READY  MESSAGE
# knowledge-graph   1m ago     False      True   successful scan: found X tags
```

#### ImagePolicy
```bash
flux get images policy knowledge-graph

# Should show:
# NAME              LATEST IMAGE                                              READY
# knowledge-graph   ...ecr...amazonaws.com/knowledge_graph_microservice:0.2.0-alpha.1-abc1234  True
```

#### HelmRelease
```bash
flux get helmreleases knowledge-graph

# Should show:
# NAME              REVISION  SUSPENDED  READY  MESSAGE
# knowledge-graph   0.2.0     False      True   Release reconciliation succeeded
```

---

### Step 6: Verify Kubernetes Deployment

```bash
# Check pods
kubectl get pods -n default -l app=verus-knowledge-graph

# Expected:
# NAME                                    READY   STATUS    RESTARTS   AGE
# verus-knowledge-graph-xxxxxxxxx-xxxxx   1/1     Running   0          2m

# Check deployment
kubectl get deployment verus-knowledge-graph -n default

# Expected:
# NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
# verus-knowledge-graph   1/1     1            1           2m

# Check service
kubectl get svc verus-knowledge-graph -n default

# Expected:
# NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
# verus-knowledge-graph   ClusterIP   10.100.x.x      <none>        80/TCP,50051/TCP

# Check ingress
kubectl get ingress verus-knowledge-graph -n default

# Expected:
# NAME                    CLASS   HOSTS                   ADDRESS                          PORTS
# verus-knowledge-graph   alb     kg.districtcyber.io     k8s-default-veruskno-xxx.elb...  80
```

---

### Step 7: Check Pod Logs

```bash
# View pod logs
kubectl logs -n default -l app=verus-knowledge-graph --tail=50

# Should see application startup logs
# Look for:
# - "Running KnowledgeGraphWeb.Endpoint"
# - No error messages
# - Health check endpoints ready
```

---

### Step 8: Test Application Endpoints

```bash
# Test health endpoint
curl https://kg.districtcyber.io/health/live

# Expected:
# {"status":"ok","timestamp":"2026-02-01T...","uptime":...}

# Test readiness
curl https://kg.districtcyber.io/health/ready

# Expected:
# {"status":"ready",...}
```

---

### Step 9: Verify Image Auto-Update (Optional)

```bash
# Make a small change and push
cd C:\Users\dmytro\repos\verus\Verus-elixir

# Make a change (e.g., add comment)
echo "# Test change" >> README.md

git add README.md
git commit -m "test: verify auto-deployment +semver: patch"
git push origin dev

# Wait for CI/CD to complete (~5 minutes)

# Check FluxCD detected new image
flux get images policy knowledge-graph

# Should show new version:
# LATEST IMAGE: ...0.2.1-alpha.1-def5678

# Check if HelmRelease updated
kubectl describe helmrelease knowledge-graph -n default | grep "Chart Version"

# Check if new pod is running
kubectl get pods -n default -l app=verus-knowledge-graph

# Should see new pod with recent creation time
```

---

## 🔍 Troubleshooting

### GitHub Actions Failed

```bash
# Check workflow logs
gh run list --repo DistrictCyber02/Verus-elixir
gh run view <run-id> --log

# Common issues:
# - Missing secrets → Add in GitHub repo settings
# - GitVersion error → Check GitVersion.yml syntax
# - ECR push failed → Check AWS credentials
```

### FluxCD Not Detecting Image

```bash
# Check ImageRepository
kubectl describe imagerepository knowledge-graph -n flux-system

# Look for errors in Events section

# Force reconciliation
flux reconcile image repository knowledge-graph

# Check ECR credentials
kubectl get secret -n flux-system
```

### ImagePolicy Not Selecting Version

```bash
# Check ImagePolicy
kubectl describe imagepolicy knowledge-graph -n flux-system

# Verify pattern matches your tags
# Pattern: ^\d+\.\d+\.\d+(-[a-zA-Z]+\.\d+)?-[a-f0-9]+$
# Should match: 0.2.0-alpha.1-abc1234

# Test pattern locally:
echo "0.2.0-alpha.1-abc1234" | grep -E '^\d+\.\d+\.\d+(-[a-zA-Z]+\.\d+)?-[a-f0-9]+$'
```

### HelmRelease Failed

```bash
# Check HelmRelease status
kubectl describe helmrelease knowledge-graph -n default

# Check events
kubectl get events -n default --sort-by='.lastTimestamp'

# View FluxCD logs
flux logs --kind=HelmRelease --name=knowledge-graph

# Common issues:
# - Invalid Helm chart → Check helm/knowledge-graph/
# - Missing values → Check helmrelease.yaml values section
# - Resource conflicts → Check if resources already exist
```

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod -n default -l app=verus-knowledge-graph

# Check events
kubectl get events -n default --field-selector involvedObject.name=<pod-name>

# Check logs
kubectl logs -n default -l app=verus-knowledge-graph

# Common issues:
# - Image pull error → Check ECR permissions
# - CrashLoopBackOff → Check application logs
# - Resource limits → Check deployment resource requests
```

---

## ✅ Success Criteria

All of the following should be true:

- ✅ GitHub Actions workflow completes successfully
- ✅ Docker image appears in ECR with correct tags
- ✅ FluxCD ImageRepository scans ECR successfully
- ✅ FluxCD ImagePolicy selects correct version
- ✅ HelmRelease deploys successfully
- ✅ Pod is running and healthy
- ✅ Service is accessible
- ✅ Ingress routes traffic correctly
- ✅ Application responds to health checks
- ✅ (Optional) Auto-update works on new commit

---

## 📊 Expected Timeline

```
0:00 - Commit pushed to GitHub
0:30 - GitHub Actions starts
2:00 - Tests complete
2:30 - GitVersion calculates version
3:00 - Docker build starts
5:00 - Docker push to ECR completes
5:30 - FluxCD detects new image (next scan cycle)
6:00 - ImagePolicy selects new version
6:30 - HelmRelease starts reconciliation
7:00 - Helm chart deployed
7:30 - Pod starts
8:00 - Pod ready and serving traffic
```

**Total time:** ~8-10 minutes from commit to deployment

---

## 🎓 What to Watch

### GitHub Actions
- GitVersion output (version number)
- Docker build logs
- ECR push confirmation

### FluxCD
- ImageRepository scan results
- ImagePolicy selected version
- HelmRelease reconciliation status

### Kubernetes
- Pod creation and startup
- Service endpoints
- Ingress configuration

---

**Created:** 2026-02-01  
**Purpose:** Verify complete GitOps workflow  
**Estimated Time:** 30 minutes
