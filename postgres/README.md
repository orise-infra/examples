# PostgreSQL Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Production Considerations](#production-considerations)
3.  [Prerequisites](#prerequisites)
4.  [Quick Start](#quick-start)
5.  [Cleanup](#cleanup)

## Overview

This example demonstrates two architectural patterns for deploying PostgreSQL:

1.  **Private Cloud (Operator-based)**: Uses the **CloudNativePG (CNPG)** operator in the `postgres-private-cloud` namespace. Best for high-availability, automated failover, and point-in-time recovery.
2.  **Edge (Native)**: Uses a standard Kubernetes **StatefulSet** in the `postgres-edge` namespace. Optimized for low resource consumption on single-node edge devices.

## Production Considerations

*   **Isolated Namespaces**: Each profile creates its own namespace (`postgres-edge` or `postgres-private-cloud`) with appropriate `orise.io` labels for multi-tenancy.
*   **Metadata & Labels**: All resources are automatically tagged with `labels` (via Kustomize) for consistent inventory management and observability.
*   **Storage Dependency**:
    *   **Private Cloud**: **Requires Longhorn** (`longhorn-ha`). The deployment command enforces a dependency on the `longhorn` Flux Kustomization.
    *   **Edge**: Uses local storage (`local-path`). **No Longhorn dependency**.
*   **Health Checks**: The deployment includes Flux health checks to validate that the StatefulSet or PostgreSQL Cluster is actually running before reporting success.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed
*   **Storage Classes** provisioned:
    *   **Private Cloud**: `longhorn-ha` (provided by Longhorn, managed by a Flux Kustomization named `longhorn`)
    *   **Edge**: `standard` (provided by Rancher's local-path-provisioner, typically the default storage class)
*   **Flux CLI** installed locally

See the [Flux Getting Started runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-getting-started.html) for Flux setup instructions.

### Verify Storage Classes

Before deploying, verify your storage classes:

```bash
kubectl get storageclass
```

**For Edge Profile**, you should see `standard` (local-path provisioner):
```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false
```

If `standard` storage class is not available, install the local-path-provisioner:
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

## Quick Start

> **Note**: This is a **development example** for testing. 
> 
> **Namespace Strategy**:
> - **Development (this example)**: Flux resources (GitRepository, Kustomization) are created in the **same namespace** as the application (`postgres-edge` or `postgres-private-cloud`). This simplifies local testing and cleanup.
> - **Production**: Flux resources should be created in the `flux-system` namespace, while applications deploy to their respective namespaces using `--target-namespace` flag.
>
> For production, use proper GitOps workflows with branch protection and automated testing.

### 1. Configure Environment

Export your repository details as environment variables.

```bash
# Set your repository URL and branch
export GIT_REPO_URL="https://github.com/orise-infra/examples" # Or your fork

# For Development (this example):
export GIT_BRANCH="example/<my branch name>"

# For Production:
# export GIT_BRANCH="main"

# Choose your profile: "edge" or "private-cloud"
export POSTGRES_PROFILE="edge" 
# export POSTGRES_PROFILE="private-cloud"

export NAMESPACE="postgres-$POSTGRES_PROFILE"
```

### 2. Create the Namespace

```bash
kubectl create namespace $NAMESPACE
```

### 3. Create GitRepository Source

**For Development (this example):**
```bash
flux create source git postgres-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=$NAMESPACE
```

**For Production:**
```bash
flux create source git postgres-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=flux-system
```

### 4. Create Kustomization

**For Development (this example):**
```bash
# Create the Kustomization in the target namespace
flux create kustomization postgres-$POSTGRES_PROFILE \
  --source=GitRepository/postgres-example \
  --path=./postgres/$POSTGRES_PROFILE \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=$NAMESPACE
```

**For Production:**
```bash
# Create the Kustomization in flux-system namespace, targeting the application namespace
flux create kustomization postgres-$POSTGRES_PROFILE \
  --source=GitRepository/postgres-example \
  --path=./postgres/$POSTGRES_PROFILE \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system \
  --target-namespace=$NAMESPACE
```

### 5. Verify the Deployment

**For Edge Profile:**
```bash
# Check StatefulSet
kubectl rollout status statefulset/edge-db -n $NAMESPACE

# Check Service
kubectl get svc edge-db -n $NAMESPACE
```
**For Private Cloud Profile:**
```bash
# Wait for Operator
kubectl wait helmrelease/cnpg -n $NAMESPACE --for=condition=Ready --timeout=10m

# Check Cluster Status
kubectl get postgresql/prod-db -n $NAMESPACE
kubectl wait postgresql/prod-db -n $NAMESPACE --for=jsonpath='{.status.currentPrimary}' --timeout=15m
```

**Common Checks:**

```bash
kubectl get pods -n $NAMESPACE
flux get kustomization postgres-$POSTGRES_PROFILE -n $NAMESPACE
```
## Cleanup

Remove the PostgreSQL example from your cluster:

```bash
flux delete kustomization postgres-$POSTGRES_PROFILE --namespace=$NAMESPACE
flux delete source git postgres-example --namespace=$NAMESPACE
kubectl delete namespace $NAMESPACE
```

