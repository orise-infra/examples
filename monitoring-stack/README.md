# Monitoring Stack Example

This example demonstrates a lightweight monitoring stack consisting of two distinct layers: **Data Collectors** and **Sinks**.

## Profiles

1.  **Edge (Native)**: Optimized for single-node or low-resource environments. Path: `./monitoring-stack/manifests`
2.  **Flux (Hybrid)**: Deploys Chronograf using Helm via Flux, while keeping the rest of the stack native. Path: `./monitoring-stack/flux/base/sinks`

## Architecture

- **Collectors**: Fluent Bit (`DaemonSet`) - Collects logs and host metrics (CPU/Mem).
- **Sinks**: 
    - **InfluxDB 1.8**: Time-series database with HTTP authentication.
    - **Chronograf**: Web interface for visualization.

## Prerequisites

- **Namespace**: `monitoring-stack`
- **Authentication**: Uses `monitoring-credentials` secret for InfluxDB.

## Deployment

This example follows the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy).

### 1. Manual Deployment (Plain Manifests)

Best for local development and rapid testing.

```bash
export TARGET_NAMESPACE="monitoring-stack"

# Create Namespace
kubectl apply -f manifests/namespace.yaml

# Create Credentials
kubectl apply -f manifests/secrets.yaml

# Deploy the Stack
kubectl apply -f manifests/influxdb.yaml
kubectl apply -f manifests/chronograf.yaml
kubectl apply -f manifests/fluent-bit.yaml
```

### 2. Flux Deployment (Chronograf only)

This method manages Chronograf as a HelmRelease through Flux.

```bash
export EXAMPLE_NAME="monitoring-stack-flux"
export TARGET_NAMESPACE="monitoring-stack"
export APP_PATH="./monitoring-stack/flux/base/sinks"
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="main"

# Ensure the native dependencies are running first
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/secrets.yaml
kubectl apply -f manifests/influxdb.yaml

# Create Flux Resources
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

## Verification

```bash
# 1. Check the status of all components
kubectl get all -n $TARGET_NAMESPACE

# 2. Check the Flux kustomization (if using Flux)
flux get kustomizations $EXAMPLE_NAME -n flux-system

# 3. Access Chronograf
kubectl port-forward svc/chronograf 8888:8888 -n $TARGET_NAMESPACE
```

## Querying Data

Once in Chronograf (`http://localhost:8888`), use these InfluxQL queries:

- **Logs**: `SELECT "time", "kubernetes_pod_name", "log" FROM "fluentbit"."autogen"."kube_logs" WHERE "log" != '' ORDER BY time DESC LIMIT 50`
- **CPU**: `SELECT mean("cpu_p") FROM "fluentbit"."autogen"."cpu.local" WHERE time > now() - 15m GROUP BY time(10s) fill(null)`
- **Mem**: `SELECT mean("Mem.used") FROM "fluentbit"."autogen"."mem.local" WHERE time > now() - 15m GROUP BY time(10s) fill(null)`

## Cleanup

```bash
# For Flux deployment
flux delete kustomization $EXAMPLE_NAME -n flux-system
flux delete source git $EXAMPLE_NAME -n flux-system

# For Manual deployment
kubectl delete -f manifests/
```
