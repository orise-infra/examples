# Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

## Deployment

```bash
export EXAMPLE_NAME="metrics-server"
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
# Check the status of the kustomization
flux get kustomizations $EXAMPLE_NAME -n flux-system

# Check the Metrics Server pods
kubectl get pods -n kube-system -l k8s-app=metrics-server

# Verify metrics are being collected
kubectl top nodes
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
```

## Notes

The `release.yaml` (inside this folder) already includes the `--kubelet-insecure-tls` argument in its spec. This is required for running Metrics Server on Kind clusters due to self-signed certificates. No extra Flux CLI flags are needed.