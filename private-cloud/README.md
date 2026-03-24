# Private Cloud Environment

This folder demonstrates a **High-Availability Deployment Profile** suitable for private cloud infrastructure.

## Characteristics
- **Multi-Node HA**: Runs workloads replicated across multiple nodes (e.g. 3 replicas).
- **Stateful**: Utilizes distributed storage via CSI drivers (Longhorn) to ensure data persists correctly across pods.
- **Operator-Driven**: Relies on Kubernetes Operators and HelmReleases (managed by Flux) to orchestrate complex lifecycles.

Currently, this environment uses **Longhorn** (Storage) and **CloudNativePG PostgreSQL** (Application) as its placeholders.

## Deployment Strategy

When deploying stateful workloads, infrastructure prerequisites must be fulfilled *first*. Here, Longhorn acts as the infrastructure layer providing the `longhorn-ha` storage class, which the Postgres Operator then consumes.

You can deploy using standard GitOps (GitRepository) or OCI Artifacts (OCIRepository). Note that if you use OCI, you push the entire `private-cloud` directory as one artifact, which contains both Longhorn and Postgres paths.

### Method 1: GitRepository (Standard)

#### 1. Deploy Longhorn Storage

```bash
export EXAMPLE_NAME="longhorn-storage"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

kubectl create namespace longhorn-system

flux create source git $EXAMPLE_NAME \
  --url=$GIT_REPO_URL \
  --branch=$GIT_BRANCH \
  --namespace=flux-system

flux create kustomization $EXAMPLE_NAME \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./private-cloud/longhorn" \
  --prune=true \
  --wait=true \
  --health-check-timeout=10m \ # Required for Longhorn init
  --namespace=flux-system
```

#### 2. Deploy Postgres Database

Deploy the operator, wait for health, and then deploy the actual 3-node HA cluster.

```bash
export EXAMPLE_NAME="postgres-ha"
export TARGET_NAMESPACE="postgres-ha"

kubectl create namespace $TARGET_NAMESPACE

# Wait for infrastructure source to sync
kubectl wait --for=condition=ready gitrepository/postgres-ha -n flux-system

# Deploy the operator first
flux create kustomization postgres-operator \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./private-cloud/postgres/base" \
  --prune=true \
  --health-check="HelmRelease/cnpg.$TARGET_NAMESPACE" \
  --namespace=flux-system

# Finally, deploy the application cluster (using longhorn-ha storage class)
flux create kustomization postgres-app \
  --source=GitRepository/$EXAMPLE_NAME \
  --path="./private-cloud/postgres/app" \
  --prune=true \
  --depends-on=postgres-operator \
  --namespace=flux-system
```

### Method 2: OCIRepository (Testing/CI)

#### 1. Push OCI Artifact and Create Source

```bash
export OCI_REPO_URL="oci://ghcr.io/orise-infra/examples/private-cloud"
export OCI_TAG="latest"
export SOURCE_NAME="private-cloud-artifacts"

# Push the entire private-cloud directory
flux push artifact $OCI_REPO_URL:$OCI_TAG \
  --path="./private-cloud" \
  --source="$(git config --get remote.origin.url)" \
  --revision="$(git branch --show-current)/$(git rev-parse HEAD)"

# Create OCI Source
flux create source oci $SOURCE_NAME \
  --url=$OCI_REPO_URL \
  --tag=$OCI_TAG \
  --namespace=flux-system
```

#### 2. Deploy infrastructure via OCI
```bash
kubectl create namespace longhorn-system

# Path is relative to the private-cloud directory we pushed
flux create kustomization longhorn-storage \
  --source=OCIRepository/$SOURCE_NAME \
  --path="./longhorn" \
  --prune=true \
  --wait=true \
  --health-check-timeout=10m \
  --namespace=flux-system
```

#### 3. Deploy App via OCI
```bash
export TARGET_NAMESPACE="postgres-ha"
kubectl create namespace $TARGET_NAMESPACE

# Operator
flux create kustomization postgres-operator \
  --source=OCIRepository/$SOURCE_NAME \
  --path="./postgres/base" \
  --prune=true \
  --health-check="HelmRelease/cnpg.$TARGET_NAMESPACE" \
  --namespace=flux-system

# Application Cluster
flux create kustomization postgres-app \
  --source=OCIRepository/$SOURCE_NAME \
  --path="./postgres/app" \
  --prune=true \
  --depends-on=postgres-operator \
  --namespace=flux-system
```
