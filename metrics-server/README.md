# Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

## Installation

This example uses Flux to deploy Metrics Server.

### Prerequisites

- A Kubernetes cluster (Kind, Minikube, etc.)
- Flux CLI installed locally

### Deployment

Deploy using Flux CLI:

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# 1. Create Source in flux-system
flux create source git metrics-server \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system

# 2. Create Kustomization in flux-system
flux create kustomization metrics-server \
  --source=GitRepository/metrics-server \
  --path=./metrics-server \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system
```

## Configuration

The `release.yaml` includes the `--kubelet-insecure-tls` argument which is required for running Metrics Server on Kind clusters due to self-signed certificates.
