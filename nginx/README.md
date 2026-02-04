# Nginx Production Example

This example demonstrates a **Production-Simulated** deployment of Nginx using Flux CD.

## Profiles

1.  **Edge**: Lightweight, single replica. Path: `./nginx/edge`
2.  **Private Cloud**: High Availability (3 replicas). Path: `./nginx/private-cloud`

## Prerequisites

*   **Kind Cluster** with Ingress support (`resources/kind-config.yaml`).
*   **HAProxy Ingress Controller** installed.
*   **GHCR Token**: Image pull secret `ghcr-secret` required in the target namespace.

## Deployment

Choose a profile (`edge` or `private-cloud`) and follow the commands below.

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
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GHCR_TOKEN \
  --docker-email=your-email@example.com \
  --namespace=$TARGET_NAMESPACE

kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}' \
  -n $TARGET_NAMESPACE

# Create Flux Resources
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

# Create Flux Resources
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