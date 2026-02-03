# OpenObserve Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Prerequisites](#prerequisites)
3.  [Quick Start](#quick-start)
4.  [Cleanup](#cleanup)

## Overview

Deploys **OpenObserve** using Flux CD.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed.

## Quick Start

> **Warning**: This is a **development example** for testing. 

### 1. Configure Environment

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export NAMESPACE="openobserve"
```

### 2. Create Namespace & Deploy

```bash
kubectl create namespace $NAMESPACE

flux create source git openobserve-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=$NAMESPACE

flux create kustomization openobserve \
  --source=GitRepository/openobserve-example \
  --path=./openobserve \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=$NAMESPACE
```

### 3. Verify

```bash
kubectl get pods -n $NAMESPACE
```

**Test Ingestion:**
```bash
kubectl port-forward svc/openobserve 5080:5080 -n $NAMESPACE
# In another terminal:
curl -u admin@example.com:password123 -XPOST http://localhost:5080/api/default/default/_json -d '[{"msg": "test"}]'
```

## Cleanup

```bash
flux delete kustomization openobserve --namespace=$NAMESPACE
flux delete source git openobserve-example --namespace=$NAMESPACE
kubectl delete namespace $NAMESPACE
```
