# PostgreSQL Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Prerequisites](#prerequisites)
3.  [Quick Start](#quick-start)
4.  [Cleanup](#cleanup)

## Overview

This example demonstrates two architectural patterns for deploying PostgreSQL:

1.  **Private Cloud (Operator-based)**: Uses the **CloudNativePG (CNPG)** operator. Best for high-availability.
2.  **Edge (Native)**: Uses a standard Kubernetes **StatefulSet**. Optimized for low resource consumption.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed.
*   **Flux CLI** installed locally.
*   **Storage Classes**:
    *   **Private Cloud**: `longhorn-ha` (provided by Longhorn).
    *   **Edge**: `standard` (local-path).

## Quick Start

> **Warning**: This is a **development example** for testing. 
> For production, Flux resources should be created in the `flux-system` namespace, and proper GitOps workflows with branch protection must be used.

### 1. Configure Environment

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export POSTGRES_PROFILE="edge" # or "private-cloud"
export NAMESPACE="postgres-$POSTGRES_PROFILE"
```

### 2. Create Namespace & Deploy

```bash
kubectl create namespace $NAMESPACE

flux create source git postgres-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=$NAMESPACE

flux create kustomization postgres-$POSTGRES_PROFILE \
  --source=GitRepository/postgres-example \
  --path=./postgres/$POSTGRES_PROFILE \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=$NAMESPACE
```

### 3. Verify

```bash
kubectl get pods -n $NAMESPACE
flux get kustomization postgres-$POSTGRES_PROFILE -n $NAMESPACE
```

## Cleanup

```bash
flux delete kustomization postgres-$POSTGRES_PROFILE --namespace=$NAMESPACE
flux delete source git postgres-example --namespace=$NAMESPACE
kubectl delete namespace $NAMESPACE
```