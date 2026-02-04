# Longhorn Distributed Storage Example

Deploys **Longhorn** as the storage engine using Flux CD.

## Prerequisites

*   **ISCSI**: Nodes must have `open-iscsi` installed.

## Local Development (Kind)

If you are using **kind**, use the provided configuration to ensure the nodes have the necessary mounts for Longhorn:

```bash
kind create cluster --config ./resources/kind-config.yaml
```

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
  
  ## Verification
  
  ```bash
  # Check the status of the kustomization
  flux get kustomizations $EXAMPLE_NAME -n flux-system
  
  # Check Longhorn pods
  kubectl get pods -n longhorn-system
  
  # Verify StorageClasses are created
  kubectl get sc | grep longhorn

  # Verify CSI Drivers are registered
  kubectl get csidrivers | grep longhorn

  ```
  
  ## Rollback

To rollback a deployment, revert the changes in your Git repository and push. Flux will automatically detect the commit change and synchronize the previous state.

```bash
git revert <commit-hash>
git push origin <branch>
```
  
  ## Cleanup
  
  ```bash
  flux delete kustomization $EXAMPLE_NAME -n flux-system
  flux delete source git $EXAMPLE_NAME -n flux-system
  kubectl delete ns longhorn-system
  ```
  