# Knowledge Graph - FluxCD Configuration

## 📋 Files

- `namespace.yaml` - Kubernetes namespace
- `gitrepository.yaml` - Source for Helm chart
- `imagerepository.yaml` - ECR image monitoring
- `imagepolicy.yaml` - Version selection (semver)
- `helmrelease.yaml` - Helm deployment
- `imageupdateautomation.yaml` - Auto-update manifests
- `kustomization.yaml` - Kustomize config

## 🔄 How It Works

```
1. ImageRepository scans ECR every 1 minute
2. ImagePolicy filters tags by pattern and selects latest version
3. ImageUpdateAutomation updates helmrelease.yaml
4. HelmRelease deploys to Kubernetes
```

## 🏷️ Tag Matching

### Pattern
```regex
^(?P<version>\d+\.\d+\.\d+(-[a-zA-Z]+\.\d+)?)-(?P<sha>[a-f0-9]+)$
```

### Supported Tags
| Branch | GitVersion | Docker Tag | Match |
|--------|-----------|------------|-------|
| main | `1.2.3` | `1.2.3-abc1234` | ✅ |
| dev | `1.3.0-alpha.5` | `1.3.0-alpha.5-abc1234` | ✅ |
| hotfix/* | `1.2.4-beta.1` | `1.2.4-beta.1-abc1234` | ✅ |
| feature/* | `1.3.0-feature.1` | `1.3.0-feature.1-abc1234` | ✅ |

### Not Supported
```
❌ latest           (no version)
❌ 1.2.3            (no SHA)
❌ v1.2.3-abc1234   (prefix 'v')
```

## 🚀 Deployment

```bash
# Apply manifests
kubectl apply -k apps/knowledge-graph/

# Check status
flux get all

# Force reconciliation
flux reconcile image repository knowledge-graph
flux reconcile helmrelease knowledge-graph
```

## 📊 Monitoring

```bash
# Check image detection
flux get images repository

# Check version selection
flux get images policy

# Check auto-updates
flux get images update

# Check deployment
kubectl get helmrelease -n default
kubectl get pods -n default -l app=verus-knowledge-graph
```

## 🔧 Configuration

### ImagePolicy
- **Range**: `>=0.1.0` (any version from 0.1.0)
- **Pattern**: Matches `VERSION-SHA` format
- **Selects**: Latest semantic version

### HelmRelease
- **Interval**: 5 minutes
- **Chart**: `./helm/knowledge-graph` from verus-elixir repo
- **Auto-update**: Via `{"$imagepolicy": "flux-system:knowledge-graph"}` marker

## 🆘 Troubleshooting

### Image not detected
```bash
# Check ImageRepository
kubectl describe imagerepository knowledge-graph -n flux-system

# Check ECR credentials
kubectl get secret -n flux-system
```

### Version not updating
```bash
# Check ImagePolicy
kubectl describe imagepolicy knowledge-graph -n flux-system

# Check pattern matches
# Pattern: ^\d+\.\d+\.\d+(-[a-zA-Z]+\.\d+)?-[a-f0-9]+$
# Example: 1.2.3-abc1234 ✅
```

### Deployment failing
```bash
# Check HelmRelease
kubectl describe helmrelease knowledge-graph -n default

# Check logs
flux logs --kind=HelmRelease --name=knowledge-graph

# Check pods
kubectl get pods -n default -l app=verus-knowledge-graph
kubectl logs -n default -l app=verus-knowledge-graph
```

---

**See also**: `Verus-elixir/GITOPS.md` for complete deployment guide
