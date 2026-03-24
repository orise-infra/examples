# Deployment Setup Guide

This guide provides common instructions for deploying the examples in this repository using Flux CD.
The repository is split into two primary environments: `/edge` and `/private-cloud`.

## Prerequisites

- A Kubernetes cluster (Kind, Minikube, or standard).
- [Flux CLI](https://fluxcd.io/flux/installation/) installed locally.
- `flux-system` namespace initialized (run `flux install` if not already done).

## Namespace Handling Strategy

See the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy) in the root README. Avoid defining the `namespace` field in Kustomization files.

## Deployment Methods

You can deploy these examples using two methods:
1. **GitRepository**: The standard GitOps flow tracking a Git branch.
2. **OCIRepository**: Packaging manifests as an OCI artifact and pushing it to a container registry. This is often easier for testing specific builds or versions.

---

### Edge Workloads

**Method 1: GitRepository (Standard)**
```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export EXAMPLE_NAME="edge-app"
export APP_PATH="./edge"
export TARGET_NAMESPACE="edge-namespace"

kubectl create namespace $TARGET_NAMESPACE

flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --namespace=flux-system
```

**Method 2: OCIRepository (Testing)**
```bash
export OCI_REPO_URL="oci://ghcr.io/orise-infra/examples/edge-app"
export OCI_TAG="latest"
export EXAMPLE_NAME="edge-app"
export TARGET_NAMESPACE="edge-namespace"

# 1. Push the artifact to your registry
flux push artifact $OCI_REPO_URL:$OCI_TAG \
  --path="./edge" \
  --source="$(git config --get remote.origin.url)" \
  --revision="$(git branch --show-current)/$(git rev-parse HEAD)"

kubectl create namespace $TARGET_NAMESPACE

# 2. Create the OCI source
flux create source oci $EXAMPLE_NAME \
  --url=$OCI_REPO_URL \
  --tag=$OCI_TAG \
  --interval=1m \
  --namespace=flux-system

# 3. Create the Kustomization (Note: path is "." since the artifact root is the edge directory)
flux create kustomization $EXAMPLE_NAME \
  --source=OCIRepository/$EXAMPLE_NAME \
  --path="." \
  --prune=true \
  --namespace=flux-system
```

---

### Private Cloud Workloads

For multi-node HA workloads requiring operators, review the dependencies. Example utilizing nested `depends-on`:

**Method 1: GitRepository (Standard)**
```bash
export TARGET_NAMESPACE="private-cloud-app"

# Base Operator
flux create kustomization app-operator \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./private-cloud/postgres/base" \
  --namespace=flux-system \
  --prune=true

# Application Cluster
flux create kustomization app-cluster \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./private-cloud/postgres/app" \
  --namespace=flux-system \
  --prune=true \
  --depends-on=app-operator
```

**Method 2: OCIRepository (Testing)**
```bash
export OCI_REPO_URL="oci://ghcr.io/orise-infra/examples/private-cloud"
export OCI_TAG="latest"
export TARGET_NAMESPACE="private-cloud-app"

# 1. Push the artifact to your registry
flux push artifact $OCI_REPO_URL:$OCI_TAG \
  --path="./private-cloud" \
  --source="$(git config --get remote.origin.url)" \
  --revision="$(git branch --show-current)/$(git rev-parse HEAD)"

# 2. Create the OCI source
flux create source oci $EXAMPLE_NAME \
  --url=$OCI_REPO_URL \
  --tag=$OCI_TAG \
  --interval=1m \
  --namespace=flux-system

# 3. Base Operator (Path is relative to the root of the private-cloud directory in the artifact)
flux create kustomization app-operator \
  --source=OCIRepository/$EXAMPLE_NAME \
  --path="./postgres/base" \
  --namespace=flux-system \
  --prune=true

# 4. Application Cluster
flux create kustomization app-cluster \
  --source=OCIRepository/$EXAMPLE_NAME \
  --path="./postgres/app" \
  --namespace=flux-system \
  --prune=true \
  --depends-on=app-operator
```

## Verification

```bash
# Check Flux resources
flux get sources -A
flux get kustomizations -A

# Check application resources
kubectl get pods,svc,ing -n $TARGET_NAMESPACE
```
