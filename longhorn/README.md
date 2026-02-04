# Longhorn Distributed Storage Example

Deploys **Longhorn** as the storage engine using Flux CD.

## Prerequisites

*   **ISCSI**: Nodes must have `open-iscsi` installed.

## Deployment

```bash
export EXAMPLE_NAME="longhorn"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# 1. Create Source
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

# 2. Create Kustomization
# Note: --health-check-timeout=10m is required as Longhorn takes time to initialize.
flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./$EXAMPLE_NAME" \
  --prune=true \
  --wait=true \
  --health-check-timeout=10m \
  --namespace=flux-system
```