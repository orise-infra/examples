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

### Manual Deployment (Plain Manifests)

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

### Flux Deployment (Chronograf only)

This method manages Chronograf as a HelmRelease through Flux. Follow the [Standard Flux Deployment Pattern](../docs/deployment-patterns.md#standard-flux-deployment-pattern) with these values:

```bash
export EXAMPLE_NAME="monitoring-stack-flux"
export TARGET_NAMESPACE="monitoring-stack"
export APP_PATH="./monitoring-stack/flux/base/sinks"
export GIT_BRANCH="main"

# Ensure the native dependencies are running first
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/secrets.yaml
kubectl apply -f manifests/influxdb.yaml

# Then follow the standard pattern from docs/deployment-patterns.md
```

### StratumOS Deployment

StratumOS uses the `flux-monitoring-stack` namespace as the default for Flux-managed monitoring resources.

```bash
export FLUX_NAMESPACE="flux-monitoring-stack"

# Create the secret in the flux-monitoring namespace
kubectl create secret generic monitoring-credentials \
  --from-literal=INFLUXDB_USERNAME=admin \
  --from-literal=INFLUXDB_PASSWORD=change-me-now \
  --namespace=$FLUX_NAMESPACE
```

## Verification

```bash
# 1. Check the status of all components
kubectl get all -n $TARGET_NAMESPACE

# 2. Check the Flux kustomization (if using Flux)
flux get kustomizations monitoring-stack-flux -n flux-system

# 3. Access Chronograf
kubectl port-forward svc/chronograf 8888:8888 -n $TARGET_NAMESPACE
```

## Querying Data

Once in Chronograf (`http://localhost:8888`), use these InfluxQL queries:

- **Logs**: `SELECT "time", "kubernetes_pod_name", "log" FROM "fluentbit"."autogen"."kube_logs" WHERE "log" != '' ORDER BY time DESC LIMIT 50`
- **CPU**: `SELECT mean("cpu_p") FROM "fluentbit"."autogen"."cpu.local" WHERE time > now() - 15m GROUP BY time(10s) fill(null)`
- **Mem**: `SELECT mean("Mem.used") FROM "fluentbit"."autogen"."mem.local" WHERE time > now() - 15m GROUP BY time(10s) fill(null)`

## Cleanup

For manual manifest cleanup:
```bash
kubectl delete -f manifests/
```

For Flux deployment cleanup, refer to [docs/deployment-patterns.md#cleanup](../docs/deployment-patterns.md#cleanup).

