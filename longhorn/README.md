# Longhorn Distributed Storage Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Prerequisites](#prerequisites)
3.  [Quick Start](#quick-start)
4.  [Cleanup](#cleanup)

## Overview

Deploys **Longhorn** as the storage engine using Flux CD.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed.
*   **ISCSSI**: Nodes must have `open-iscsi` installed.

## Quick Start

> **Warning**: This is a **development example** for testing. 

### 1. Configure Environment

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export NAMESPACE="longhorn-system"
```

### 2. Create Namespace & Deploy

```bash
kubectl create namespace $NAMESPACE

flux create source git longhorn-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=$NAMESPACE

flux create kustomization longhorn \
  --source=GitRepository/longhorn-example \
  --path=./longhorn \
  --prune=true \
  --wait=true \
  --interval=10m \
  --health-check-timeout=10m \
  --namespace=$NAMESPACE
```

### 3. Verify

```bash
kubectl get pods -n $NAMESPACE
kubectl get sc
```

## Cleanup

```bash
flux delete kustomization longhorn --namespace=$NAMESPACE
flux delete source git longhorn-example --namespace=$NAMESPACE
kubectl delete namespace $NAMESPACE
```
