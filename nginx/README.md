# Nginx Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Prerequisites](#prerequisites)
3.  [Quick Start](#quick-start)
4.  [Cleanup](#cleanup)

## Overview

Simple **Nginx** deployment using Flux CD.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed.

## Quick Start

> **Warning**: This is a **development example** for testing. 
> For production, Flux resources should be created in the `flux-system` namespace.

### 1. Configure Environment

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"
export NAMESPACE="nginx-flux-example"
```

### 2. Create Namespace & Deploy

```bash
kubectl create namespace $NAMESPACE

flux create source git nginx-example \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --interval=1m \
  --namespace=$NAMESPACE

flux create kustomization nginx \
  --source=GitRepository/nginx-example \
  --path=./nginx \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=$NAMESPACE
```

### 3. Verify

```bash
kubectl get pods -n $NAMESPACE
kubectl get svc nginx-service -n $NAMESPACE
```

## Cleanup

```bash
flux delete kustomization nginx --namespace=$NAMESPACE
flux delete source git nginx-example --namespace=$NAMESPACE
kubectl delete namespace $NAMESPACE
```