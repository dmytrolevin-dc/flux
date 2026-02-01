# Deployment

This service is deployed using **GitOps** with FluxCD.

## 📚 Documentation

All deployment documentation is centralized in the **flux repository**:

- **[GitOps Deployment Guide](https://github.com/dmytrolevin-dc/flux/blob/main/docs/GITOPS.md)** - Complete deployment guide
- **[Setup Summary](https://github.com/dmytrolevin-dc/flux/blob/main/docs/SETUP_COMPLETE.md)** - What was created and next steps
- **[FluxCD Configuration](https://github.com/dmytrolevin-dc/flux/blob/main/apps/knowledge-graph/README.md)** - FluxCD manifests guide

## 🚀 Quick Deploy

```bash
# 1. Commit your changes
git add .
git commit -m "feat: your changes +semver: minor"
git push origin dev

# 2. CI/CD will automatically:
#    - Run tests
#    - Calculate version with GitVersion
#    - Build Docker image
#    - Push to ECR
#    - FluxCD will auto-deploy to Kubernetes
```

## 🔧 Configuration Files

- `GitVersion.yml` - Semantic versioning configuration
- `.github/workflows/ci.yml` - CI/CD pipeline
- `helm/knowledge-graph/` - Helm chart for Kubernetes deployment

## 📊 Versioning

This project uses **GitVersion** for automatic semantic versioning:

```bash
# Patch bump (0.1.0 → 0.1.1)
git commit -m "fix: bug fix +semver: patch"

# Minor bump (0.1.1 → 0.2.0)
git commit -m "feat: new feature +semver: minor"

# Major bump (0.2.0 → 1.0.0)
git commit -m "feat!: breaking change +semver: major"
```

**Branch-based versions:**
- `main` → `1.2.3` (stable)
- `dev` → `1.3.0-alpha.5` (development)
- `feature/*` → `1.3.0-feature.1` (feature branches)
- `hotfix/*` → `1.2.4-beta.1` (hotfixes)

## 🔍 Monitoring

```bash
# Check deployment status
kubectl get pods -n default -l app=verus-knowledge-graph

# Check service health
curl https://kg.districtcyber.io/health/live

# View logs
kubectl logs -n default -l app=verus-knowledge-graph --tail=100
```

---

For detailed deployment documentation, see the **[flux repository](https://github.com/dmytrolevin-dc/flux)**.
