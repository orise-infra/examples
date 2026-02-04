# OpenObserve Example

Deploys **OpenObserve** using Flux CD.

## Deployment

```bash
export EXAMPLE_NAME="openobserve"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# 1. Create Source
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

# 2. Create Kustomization
flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./$EXAMPLE_NAME" \
  --prune=true \
  --namespace=flux-system
```

## Verification

```bash
# Forward port to access the UI/API
kubectl port-forward svc/openobserve 5080:5080 -n openobserve

# Test ingestion
curl -u admin@example.com:password123 -XPOST http://localhost:5080/api/default/default/_json -d '[{"msg": "test"}]'
```

## Rollback

To rollback a deployment, revert the changes in your Git repository and push. Flux will automatically detect the commit change and synchronize the previous state.

```bash
git revert <commit-hash>
git push origin <branch>
```

## Cleanup

To remove the example from your cluster:

```bash
flux delete kustomization $EXAMPLE_NAME -n flux-system
flux delete source git $EXAMPLE_NAME -n flux-system
# Optionally delete the namespace
kubectl delete ns openobserve
```
