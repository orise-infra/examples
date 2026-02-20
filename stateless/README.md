# Nginx Production Example

This example demonstrates a **Production-Simulated** deployment of Nginx using Flux CD and Helm Charts.

## Profiles

1.  **Edge**: Lightweight, single replica. Path: `./nginx/edge`
2.  **Private Cloud**: High Availability (3 replicas). Path: `./nginx/private-cloud`

## Prerequisites

*   **Kind Cluster** with Ingress support (`resources/kind-config.yaml`).
*   **HAProxy Ingress Controller** installed.
*   **GHCR Token**: Image pull secret `ghcr-secret` required in the target namespace.

## Deployment

Choose a profile (`edge` or `private-cloud`) and follow the commands below.

**Note:** This example follows the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy). Flux resources go in `flux-system`, application resources in their own namespace.


### 1. Edge Profile

```bash
export EXAMPLE_NAME="nginx-edge"
export TARGET_NAMESPACE="nginx-edge"
export APP_PATH="./nginx/edge"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Create Namespace and Secret first
kubectl create namespace $TARGET_NAMESPACE
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=hurtadosanti \
  --docker-password=${GHCR_TOKEN} \
  --docker-email=hurtadosanti@proton.me \
  --namespace=$TARGET_NAMESPACE

kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}' \
  -n $TARGET_NAMESPACE

# Create Flux Resources (in flux-system namespace)
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --namespace=flux-system
```

### 2. Private Cloud Profile

```bash
export EXAMPLE_NAME="nginx-private-cloud"
export TARGET_NAMESPACE="nginx-private-cloud"
export APP_PATH="./nginx/private-cloud"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Create Namespace and Secret first
kubectl create namespace $TARGET_NAMESPACE
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GHCR_TOKEN \
  --docker-email=your-email@example.com \
  --namespace=$TARGET_NAMESPACE

kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}' \
  -n $TARGET_NAMESPACE

# Create Flux Resources (in flux-system namespace)
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --namespace=flux-system
```

## Verification

```bash
# Check the Flux Kustomization status (in flux-system)
flux get kustomizations $EXAMPLE_NAME -n flux-system

# Check the Helm Release (in app namespace)
flux get helmreleases $EXAMPLE_NAME -n $TARGET_NAMESPACE

# Check the Nginx pods (in app namespace)
kubectl get pods -n $TARGET_NAMESPACE

# Test access (assuming you have an Ingress controller)
curl -H "Host: nginx.example.com" http://localhost
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
# Delete the namespace and the ghcr-secret
kubectl delete ns $TARGET_NAMESPACE
```
