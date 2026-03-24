# Edge Environment

This folder demonstrates a low resource footprint deployment suitable for edge devices or resource-constrained nodes. 

## Characteristics
- **Single Node**: Operates as a simple deployment without clustering logic.
- **Stateless**: Uses standard, local storage without requiring distributed storage solutions.
- **Pure Manifests**: Uses standard Kubernetes manifests (`deployment`, `service`, `ingress`) instead of Helm.

Currently, this environment uses **Nginx** as its placeholder application.

## Deployment

You can deploy this via Flux using standard GitOps (GitRepository) or by packaging it as an OCI artifact for easier testing.

### Method 1: GitRepository (Standard)

```bash
export EXAMPLE_NAME="edge-app"
export TARGET_NAMESPACE="edge-app"
export APP_PATH="./edge"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Create Namespace
kubectl create namespace $TARGET_NAMESPACE

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

### Method 2: OCIRepository (Testing/CI)

This method packages the `./edge` folder into a container image and pushes it to a registry, which Flux then downloads.

```bash
export EXAMPLE_NAME="edge-app"
export TARGET_NAMESPACE="edge-app"
export OCI_REPO_URL="oci://ghcr.io/orise-infra/examples/edge-app"
export OCI_TAG="latest"

# 1. Push artifact to registry
flux push artifact $OCI_REPO_URL:$OCI_TAG \
  --path="./edge" \
  --source="$(git config --get remote.origin.url)" \
  --revision="$(git branch --show-current)/$(git rev-parse HEAD)"

# 2. Create Namespace
kubectl create namespace $TARGET_NAMESPACE

# 3. Create OCI Source
flux create source oci $EXAMPLE_NAME \
  --url=$OCI_REPO_URL \
  --tag=$OCI_TAG \
  --namespace=flux-system

# 4. Create Kustomization (Note: path is "." since the artifact root is the edge directory)
flux create kustomization $EXAMPLE_NAME \
  --source=OCIRepository/$EXAMPLE_NAME \
  --path="." \
  --prune=true \
  --namespace=flux-system
```
