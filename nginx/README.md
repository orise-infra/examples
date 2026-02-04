# Nginx Production Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Prerequisites](#prerequisites)
3.  [Quick Start](#quick-start)
4.  [Cleanup](#cleanup)

## Overview

This example demonstrates a **Production-Simulated** deployment of Nginx using Flux CD. Unlike other examples in this repository, this setup intentionally uses the `flux-system` namespace to manage resources, mirroring a real-world GitOps control plane.

Two profiles are provided:

1.  **Edge**: Lightweight, single replica, custom "Edge" landing page.
2.  **Private Cloud**: High Availability (3 replicas), custom "Private Cloud" landing page.

Both profiles use **HAProxy Ingress** and mount a custom HTML page via ConfigMap.

## Prerequisites

*   **Flux CLI** installed locally.
*   **Kind Cluster** with Ingress support.
    *   Use the shared config: `resources/kind-config.yaml`
    *   `kind create cluster --config resources/kind-config.yaml`
*   **HAProxy Ingress Controller**:
    *   Must be installed in the cluster.
    *   [Installation Guide](https://github.com/haproxytech/kubernetes-ingress#installation) or use Helm:
        ```bash
        helm repo add haproxytech https://haproxytech.github.io/helm-charts
        helm install haproxy-kubernetes-ingress haproxytech/kubernetes-ingress \
          --create-namespace --namespace haproxy-controller \
          --set controller.service.type=NodePort \
          --set controller.service.nodePorts.http=30080 \
          --set controller.service.nodePorts.https=30443
        ```
    *   *Note: The Kind config maps host ports 8080/8443 to container ports 30080/30443, so ensure your Ingress Controller aligns with those or use `NodePort` mapping if preferred.*
*   **GHCR Token**: A GitHub Personal Access Token (PAT) with `read:packages` scope is required to pull the container image.

## Quick Start

> **Production Note**: This example strictly follows the **Production** pattern where Flux resources live in `flux-system` and target the application namespace.

### 1. Configure Environment

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Choose Profile: "edge" or "private-cloud"
export PROFILE="edge" 
# export PROFILE="private-cloud"

export TARGET_NAMESPACE="nginx-$PROFILE"
```

### 2. Create Namespace & Deploy (Production Style)

Create the target application namespace manually (simulating tenant separation):

```bash
kubectl create namespace $TARGET_NAMESPACE
```

Create the image pull secret in the target namespace and patch the default ServiceAccount to use it. This avoids modifying the Deployment manifest.

```bash
# 1. Create the secret
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GHCR_TOKEN \
  --docker-email=your-email@example.com \
  --namespace=$TARGET_NAMESPACE

# 2. Patch the default ServiceAccount to use this secret
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}' \
  -n $TARGET_NAMESPACE
```

Deploy using Flux, keeping the management in `flux-system`:

```bash
# 1. Create Source in flux-system
flux create source git nginx-repo \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system

# 2. Create Kustomization in flux-system, targeting the App Namespace
flux create kustomization nginx-$PROFILE \
  --source=GitRepository/nginx-repo \
  --path=./nginx/$PROFILE \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system \
  --target-namespace=$TARGET_NAMESPACE
```

### 3. Verify

Check the deployment:

```bash
kubectl get pods -n $TARGET_NAMESPACE
kubectl get ingress -n $TARGET_NAMESPACE
```

**Access the Application:**
If using Kind with the provided config and HAProxy Ingress listening on port 80:

```bash
# Curl localhost (mapped to Kind node port 8080)
curl http://localhost:8080/
```
You should see the profile-specific HTML content (e.g., "Welcome to ORISE Edge Nginx").

## Cleanup

```bash
flux delete kustomization nginx-$PROFILE --namespace=flux-system
flux delete source git nginx-repo --namespace=flux-system
kubectl delete namespace $TARGET_NAMESPACE
```
