# Deployment Setup Guide

This guide provides common instructions for deploying the examples in this repository using Flux CD.

## Prerequisites

- A Kubernetes cluster (Kind, Minikube, or any standard cluster).
- [Flux CLI](https://fluxcd.io/flux/installation/) installed locally.
- (Optional) [Kind](https://kind.sigs.k8s.io/) installed for local testing.

## Common Environment Variables

Most examples require these basic variables:

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export EXAMPLE_NAME="<example-folder-name>"
export APP_PATH="./$EXAMPLE_NAME"
```

## Deployment

We use the `flux-system` namespace for all Flux resources (Source and Kustomization). The application's specific namespace is defined within its own manifests (e.g., `namespace.yaml`).

```bash
# 1. Create Source
# We create a unique source for the example to allow specific branches if needed.
flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system

# 2. Create Kustomization
flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system
```

## Verification

Common commands to verify the deployment:

```bash
flux get kustomizations -n flux-system
# Check the application pods (replace <namespace> with the example's actual namespace)
kubectl get pods -n <namespace>
```

## Cleanup

```bash
flux delete kustomization $EXAMPLE_NAME --namespace=flux-system
flux delete source git $EXAMPLE_NAME --namespace=flux-system
# The application namespace should be deleted manually
kubectl delete namespace <namespace>
```
