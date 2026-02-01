# FluxCD GitOps Repository

Central repository for Kubernetes deployments using FluxCD and GitOps principles.

## 📁 Structure

```
flux/
├── docs/                          # 📚 Centralized documentation
│   ├── GITOPS.md                 # Complete GitOps deployment guide
│   └── SETUP_COMPLETE.md         # Setup summary and next steps
│
└── apps/                          # 🚀 Application deployments
    └── knowledge-graph/
        ├── README.md              # FluxCD configuration guide
        ├── namespace.yaml
        ├── gitrepository.yaml
        ├── imagerepository.yaml
        ├── imagepolicy.yaml
        ├── helmrelease.yaml
        ├── imageupdateautomation.yaml
        └── kustomization.yaml
```

## 📚 Documentation

### General Guides
- **[GitOps Deployment Guide](docs/GITOPS.md)** - Complete guide to GitOps workflow
- **[Setup Summary](docs/SETUP_COMPLETE.md)** - What was created and how to use it
- **[Quick Test](docs/QUICK_TEST.md)** - ⭐ Fast workflow verification (5 commands)
- **[Testing Checklist](docs/TESTING_CHECKLIST.md)** - Detailed testing and troubleshooting

### Application-Specific
- **[Knowledge Graph](apps/knowledge-graph/README.md)** - FluxCD configuration for knowledge-graph service

## 🎯 How It Works

```
Developer commits code
    ↓
GitHub Actions (CI/CD)
    ├─ Test
    ├─ GitVersion (semantic versioning)
    ├─ Build Docker image
    └─ Push to ECR
        ↓
FluxCD (Kubernetes)
    ├─ ImageRepository (scans ECR)
    ├─ ImagePolicy (selects version)
    ├─ ImageUpdateAutomation (updates manifest)
    └─ HelmRelease (deploys)
        ↓
    ✅ Running in Kubernetes
```

## 🚀 Quick Start

### 1. Install FluxCD
```bash
flux install --components-extra=image-reflector-controller,image-automation-controller
```

### 2. Apply Application Manifests
```bash
# Deploy knowledge-graph
kubectl apply -k apps/knowledge-graph/

# Check status
flux get all
```

### 3. Monitor Deployments
```bash
# Watch FluxCD reconciliation
flux get images repository
flux get images policy
flux get helmreleases

# Check application pods
kubectl get pods -n default
```

## 📊 Deployed Applications

| Application | Namespace | Status | Documentation |
|-------------|-----------|--------|---------------|
| **knowledge-graph** | `default` | ✅ Active | [README](apps/knowledge-graph/README.md) |

## 🔧 FluxCD Components

### Image Automation
- **ImageRepository** - Scans container registries (ECR)
- **ImagePolicy** - Selects image versions (semver)
- **ImageUpdateAutomation** - Updates manifests automatically

### Deployment
- **GitRepository** - Source for Helm charts
- **HelmRelease** - Deploys Helm charts
- **Kustomization** - Applies Kubernetes manifests

## 🎓 Key Concepts

### GitOps Principles
1. **Declarative** - Desired state in Git
2. **Versioned** - All changes tracked
3. **Immutable** - Git as source of truth
4. **Automated** - Continuous reconciliation

### Semantic Versioning
- `1.2.3` - Production (main branch)
- `1.3.0-alpha.5` - Development (dev branch)
- `1.2.4-beta.1` - Hotfix (hotfix/* branches)
- `1.3.0-feature.1` - Feature (feature/* branches)

### Image Tagging
```
Format: VERSION-SHA
Examples:
  ✅ 1.2.3-abc1234
  ✅ 1.3.0-alpha.5-def5678
  ❌ latest (not used in production)
```

## 🔍 Monitoring & Troubleshooting

### Check FluxCD Status
```bash
# Overall status
flux get all

# Specific components
flux get images repository
flux get images policy
flux get helmreleases

# Logs
flux logs --kind=HelmRelease --name=knowledge-graph
```

### Force Reconciliation
```bash
# Force image scan
flux reconcile image repository knowledge-graph

# Force deployment
flux reconcile helmrelease knowledge-graph
```

### Common Issues
See [GitOps Guide - Troubleshooting](docs/GITOPS.md#troubleshooting)

## 🌍 Multi-Environment Strategy

### Current: Single Cluster, Single Namespace
```
Kubernetes Cluster
└── namespace: default
    └── knowledge-graph (dev/alpha versions)
```

### Future: Multi-Namespace
```
Kubernetes Cluster
├── namespace: production
│   └── knowledge-graph (stable versions)
└── namespace: staging
    └── knowledge-graph (alpha versions)
```

See [GitOps Guide](docs/GITOPS.md) for multi-environment setup.

## 📝 Adding New Applications

1. Create application directory: `apps/your-app/`
2. Add FluxCD manifests (see `apps/knowledge-graph/` as template)
3. Update this README
4. Apply: `kubectl apply -k apps/your-app/`

## 🔐 Security

- AWS credentials managed via IRSA (IAM Roles for Service Accounts)
- Secrets stored in Kubernetes Secrets
- Image scanning in ECR
- RBAC for FluxCD controllers

## 📚 References

- [FluxCD Documentation](https://fluxcd.io/)
- [GitOps Principles](https://opengitops.dev/)
- [Semantic Versioning](https://semver.org/)
- [Helm](https://helm.sh/)

---

**Maintained by**: DevOps Team  
**Last Updated**: 2026-02-01  
**FluxCD Version**: v2.x
