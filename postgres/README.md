# PostgreSQL Example

This example demonstrates two architectural patterns for deploying PostgreSQL.

## Profiles

1.  **Private Cloud (Operator-based)**: Uses the **CloudNativePG (CNPG)** operator. Path: `./postgres/private-cloud`
2.  **Edge (Native)**: Uses a standard Kubernetes **StatefulSet**. Path: `./postgres/edge`

## Prerequisites

*   **Storage Classes**:
    *   **Private Cloud**: `longhorn-ha` (provided by Longhorn).
    *   **Edge**: `standard` (local-path).

## Deployment

Choose a profile (`edge` or `private-cloud`) and follow the commands below.

### 1. Edge Profile (Native StatefulSet)

```bash
export EXAMPLE_NAME="postgres-edge"
export APP_PATH="./postgres/edge"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --namespace=flux-system
```

### 2. Private Cloud Profile (Operator-based)

```bash
export EXAMPLE_NAME="postgres-private-cloud"
export APP_PATH="./postgres/private-cloud"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path=$APP_PATH \
  --prune=true \
  --namespace=flux-system
```
