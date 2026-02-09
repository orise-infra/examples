# Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

## Deployment

This example follows the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy).

```bash
export EXAMPLE_NAME="metrics-server"
export TARGET_NAMESPACE="metrics-server"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# 1. Create Namespace
kubectl create namespace $TARGET_NAMESPACE

# 2. Create Source (in flux-system)
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

# 3. Create Kustomization (in flux-system)
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
kubectl get pods -n $TARGET_NAMESPACE

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
kubectl delete ns $TARGET_NAMESPACE
```

## Notes

The `release.yaml` (inside this folder) already includes the `--kubelet-insecure-tls` argument in its spec. This is required for running Metrics Server on Kind clusters due to self-signed certificates. No extra Flux CLI flags are needed.
