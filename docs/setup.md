# Deployment Setup Guide

This guide provides common instructions for deploying the examples in this repository using Flux CD.

## Prerequisites

- A Kubernetes cluster (Kind, Minikube, or any standard cluster).
- [Flux CLI](https://fluxcd.io/flux/installation/) installed locally.
- (Optional) [Kind](https://kind.sigs.k8s.io/) installed for local testing.
- `flux-system` namespace initialized (run `flux install` if not already done).

## Namespace Handling Strategy

See the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy) in the root README.

### Common Environment Variables

Most examples require these basic variables:

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export EXAMPLE_NAME="<example-folder-name>"
export APP_PATH="./$EXAMPLE_NAME"
export TARGET_NAMESPACE="<application-namespace>"
```

### Deployment Steps

```bash
# 1. Create the Application Namespace (if not auto-created)
# Note: This may be created automatically by the kustomization, but it's safer to pre-create it.
kubectl create namespace $TARGET_NAMESPACE

# 2. Create the Flux Source (in flux-system)
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system

# 3. Create the Flux Kustomization (in flux-system, targets app namespace)
flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system
```

## Verification

```bash
# Check Flux resources (in flux-system)
flux get sources -n flux-system
flux get kustomizations -n flux-system

# Check application resources (in app namespace)
kubectl get pods -n $TARGET_NAMESPACE
kubectl get all -n $TARGET_NAMESPACE
```

## Cleanup

```bash
# Delete Flux resources (from flux-system)
flux delete kustomization $EXAMPLE_NAME -n flux-system
flux delete source git $EXAMPLE_NAME -n flux-system

# Delete the application namespace
kubectl delete namespace $TARGET_NAMESPACE
```
