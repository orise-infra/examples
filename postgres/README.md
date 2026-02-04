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

This profile requires a two-step deployment to ensure the **CloudNativePG** operator is ready before the database cluster is created.

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Step 1: Deploy the Operator (Infrastructure)
# Ensure the source is created and ready before proceeding
flux create source git postgres-infra \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

# Wait for the source to be reconciled
kubectl wait --for=condition=ready gitrepository/postgres-infra -n flux-system

# Deploy the operator and wait for the HelmRelease to be healthy
flux create kustomization postgres-operator \
  --source=GitRepository/postgres-infra \
  --path="./postgres/private-cloud/base" \
  --prune=true \
  --health-check="HelmRelease/cnpg.flux-system" \
  --namespace=flux-system

# Step 2: Deploy the Cluster (Application)
# The operator webhook must be ready for this to pass dry-run validation
flux create kustomization postgres-app \
  --source=GitRepository/postgres-infra \
  --path="./postgres/private-cloud/app" \
  --prune=true \
  --depends-on=postgres-operator \
  --namespace=flux-system
```

## Verification

```bash
# 1. Check the status of the kustomizations
# For Edge, use 'postgres-edge'. For Private Cloud, check both 'postgres-operator' and 'postgres-app'.
flux get kustomizations -n flux-system

# 2. Check the PostgreSQL pods and PersistentVolumeClaims (PVCs)
# Replace <namespace> with 'postgres-edge' or 'postgres-private-cloud'
kubectl get pods,pvc -n <namespace>

# 3. Verify Persistent Volumes (PVs) are provisioned and bound
# This confirms the storage class (longhorn-ha or standard) is working.
kubectl get pv | grep <namespace>

# 4. (Optional) For Private Cloud, check the CNPG Cluster status
kubectl get cluster -n postgres-private-cloud
```

## Troubleshooting

### Private Cloud: "no matches for kind Cluster in version postgresql.cnpg.io/v1"

If you see this reconciliation error, it means the CNPG operator hasn't been fully deployed yet when Flux attempts to apply the Cluster resource.

**Resolution**: This is automatically handled by the Kustomization configuration:
- The `cluster.yaml` includes a dependency annotation (`kustomize.toolkit.fluxcd.io/depends-on`) that tells Flux to wait for the `cnpg` HelmRelease to be ready before deploying the Cluster resource
- The `operator.yaml` (HelmRelease) is configured with `install.crds: Create` and `upgrade.crds: CreateReplace` to ensure CRDs are installed
- The `source.yaml` (HelmRepository) is deployed in the `flux-system` namespace for proper Flux integration

Allow 2-3 minutes for the CNPG operator to fully deploy before the Cluster resource is created. Check the status with:

```bash
# Check HelmRelease status
kubectl get helmreleases -n flux-system | grep cnpg

# Check for any errors
kubectl describe helmrelease cnpg -n flux-system
```

## Rollback

To rollback a deployment, revert the changes in your Git repository and push. Flux will automatically detect the commit change and synchronize the previous state.

```bash
git revert <commit-hash>
git push origin <branch>
```

## Cleanup

To remove the example from your cluster:

```bash
flux delete kustomization $EXAMPLE_NAME -n flux-system
flux delete source git $EXAMPLE_NAME -n flux-system
# Optionally delete the namespace
kubectl delete ns <namespace>
```
