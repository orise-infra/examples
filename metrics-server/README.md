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

## Notes

The `release.yaml` (inside this folder) already includes the `--kubelet-insecure-tls` argument in its spec. This is required for running Metrics Server on Kind clusters due to self-signed certificates. No extra Flux CLI flags are needed.