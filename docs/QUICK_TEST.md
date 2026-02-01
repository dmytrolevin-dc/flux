# Quick Test - GitOps Workflow

## 🚀 Fast Track (5 commands)

```bash
# 1. Commit and push
cd C:\Users\dmytro\repos\verus\Verus-elixir
git add GitVersion.yml .github/workflows/ci.yml helm/
git commit -m "feat: add GitOps deployment +semver: minor"
git push origin dev

# 2. Apply FluxCD manifests
cd C:\Users\dmytro\repos\flux
git add apps/knowledge-graph/ docs/
git commit -m "Add knowledge-graph FluxCD automation"
git push origin main
kubectl apply -k apps/knowledge-graph/

# 3. Watch deployment
kubectl get pods -n default -l app=verus-knowledge-graph -w

# 4. Check FluxCD
flux get all

# 5. Test endpoint
curl https://kg.districtcyber.io/health/live
```

---

## ✅ Quick Checks

### 1. GitHub Actions (2 min)
```bash
# https://github.com/DistrictCyber02/Verus-elixir/actions
# Should see: ✅ All checks passed
```

### 2. ECR Image (3 min)
```bash
aws ecr list-images --repository-name knowledge_graph_microservice --region us-east-2 | grep imageTag
# Should see: 0.2.0-alpha.X-XXXXXXX
```

### 3. FluxCD Status (5 min)
```bash
flux get images policy knowledge-graph
# Should show: LATEST IMAGE with your version
```

### 4. Pod Running (7 min)
```bash
kubectl get pods -n default -l app=verus-knowledge-graph
# Should show: 1/1 Running
```

### 5. Service Working (8 min)
```bash
curl https://kg.districtcyber.io/health/live
# Should return: {"status":"ok",...}
```

---

## 🔴 If Something Fails

### GitHub Actions Failed
```bash
gh run list --repo DistrictCyber02/Verus-elixir
gh run view --log
```

### FluxCD Not Working
```bash
flux logs --all-namespaces
kubectl describe helmrelease knowledge-graph -n default
```

### Pod Not Starting
```bash
kubectl describe pod -n default -l app=verus-knowledge-graph
kubectl logs -n default -l app=verus-knowledge-graph
```

---

## 📊 Expected Result

```
✅ GitHub Actions: Success (2-3 min)
✅ ECR Image: Pushed (3-5 min)
✅ FluxCD: Detected (5-6 min)
✅ Pod: Running (7-8 min)
✅ Service: Responding (8-10 min)
```

**Total:** ~10 minutes from commit to working service

---

See `TESTING_CHECKLIST.md` for detailed troubleshooting.
