# GitOps Deployment Guide

## 🚀 Quick Start

### 1. Commit Changes
```bash
# Verus-elixir repo
git add GitVersion.yml .github/workflows/ci.yml helm/
git commit -m "feat: add GitOps deployment with Helm and GitVersion"
git push origin dev

# Flux repo
cd ../flux
git add apps/knowledge-graph/
git commit -m "Add knowledge-graph FluxCD automation"
git push origin main
```

### 2. Setup GitHub Secrets
In Verus-elixir repository settings, add:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `FLUX_REPO_TOKEN` (GitHub PAT with repo scope)

### 3. Install FluxCD Image Automation
```bash
flux install --components-extra=image-reflector-controller,image-automation-controller
```

### 4. Apply FluxCD Manifests
```bash
kubectl apply -k apps/knowledge-graph/
flux get all
```

### 5. Trigger First Build
```bash
git commit --allow-empty -m "feat: trigger initial deployment"
git push origin dev
```

---

## 📊 How It Works

### Workflow
```
Developer commits → GitVersion calculates version → CI/CD builds image
→ Push to ECR → FluxCD detects → Auto-deploys to Kubernetes
```

### Version Examples
```bash
# On main branch
git commit -m "feat: new feature +semver: minor"
→ Version: 1.2.0
→ Docker: 1.2.0-abc1234

# On dev branch  
git commit -m "feat: new feature +semver: minor"
→ Version: 1.2.0-alpha.5
→ Docker: 1.2.0-alpha.5-abc1234
```

---

## 🔧 Configuration Files

### GitVersion (`GitVersion.yml`)
- Automatic semantic versioning
- Branch-based versions (main: stable, dev: alpha)

### CI/CD (`.github/workflows/ci.yml`)
- Test → GitVersion → Build → Push to ECR
- Multi-tag strategy: `VERSION-SHA`, `VERSION`, `latest`

### Helm Chart (`helm/knowledge-graph/`)
- Deployment, Service, Ingress templates
- Configurable via `values.yaml`

### FluxCD (`flux/apps/knowledge-graph/`)
- `imagerepository.yaml` - Monitors ECR
- `imagepolicy.yaml` - Selects version by semver
- `helmrelease.yaml` - Deploys Helm chart
- `imageupdateautomation.yaml` - Auto-updates manifests

---

## 📝 FluxCD Tag Matching

### Supported Tags
```
✅ 1.2.3-abc1234           (production)
✅ 1.3.0-alpha.5-abc1234   (dev)
✅ 1.2.4-beta.1-abc1234    (hotfix)
❌ latest                  (no version)
❌ 1.2.3                   (no SHA)
```

### Pattern
```regex
^(?P<version>\d+\.\d+\.\d+(-[a-zA-Z]+\.\d+)?)-(?P<sha>[a-f0-9]+)$
```

---

## 🎯 Version Bumping

```bash
# Patch (0.1.0 → 0.1.1)
git commit -m "fix: bug fix +semver: patch"

# Minor (0.1.1 → 0.2.0)
git commit -m "feat: new feature +semver: minor"

# Major (0.2.0 → 1.0.0)
git commit -m "feat!: breaking change +semver: major"
```

---

## 🔍 Monitoring

```bash
# Check FluxCD status
flux get all

# Check deployment
kubectl get pods -n default -l app=verus-knowledge-graph

# Test endpoint
curl https://kg.districtcyber.io/health/live
```

---

## 📚 Additional Documentation

- **GitVersion**: See `GitVersion.yml` configuration
- **Helm Chart**: See `helm/knowledge-graph/values.yaml`
- **FluxCD**: See `flux/apps/knowledge-graph/README.md`

---

## 🆘 Troubleshooting

### Image not updating
```bash
flux reconcile image repository knowledge-graph
flux reconcile image policy knowledge-graph
```

### HelmRelease failing
```bash
flux logs --kind=HelmRelease --name=knowledge-graph
kubectl describe helmrelease knowledge-graph -n default
```

---

**Created**: 2026-02-01  
**Status**: Production Ready
