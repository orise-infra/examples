# OpenObserve Example (Flux)

This example demonstrates how to deploy OpenObserve using Flux CD with a fully isolated, namespace-scoped configuration.

## Prerequisites

*   Kubernetes Cluster with **Flux CD** installed.

## Deployment Strategy

This example uses a **Tenant** deployment model rather than `flux bootstrap`:
1.  **Pre-installed Flux**: Assumes Flux is already running on the cluster (bootstrapped elsewhere).
2.  **Namespace Isolation**: All resources are confined to the `openobserve` namespace.
3.  **Read-Only Git**: Uses a public HTTP URL for the `GitRepository`, avoiding the need for Git write credentials (PAT) which `flux bootstrap` requires.

## Quick Start

**All commands must be executed from the `openobserve/` directory.**

```bash
cd openobserve
```

### 1. Bootstrap Namespace
Create the namespace first. This allows the Flux resources to be fully contained within the tenant namespace.

```bash
kubectl apply -f namespace.yaml
```

### 2. Deploy via Flux
Apply the Flux delivery manifest to the `openobserve` namespace.

```bash
kubectl apply -f flux-delivery.yaml -n openobserve
```

### 3. Verify Deployment
Check the Flux reconciliation status and OpenObserve deployment.

```bash
flux get kustomization openobserve -n openobserve
flux get helmrelease -n openobserve
kubectl get pods -n openobserve
```

### 4. Get Ingestion Credentials
For this example, we use the root user credentials for ingestion.
*   **Username:** `admin@example.com`
*   **Password:** `password123` (defined in `secret.yaml`)

You can use these credentials to configure Fluent Bit or any other agent.

### 5. Verify Ingestion
You can verify that OpenObserve is accepting logs by sending a test entry.

First, port-forward the OpenObserve service:
```bash
kubectl port-forward svc/openobserve 5080:5080 -n openobserve
```

Then, in a separate terminal, send a log entry:

```bash
curl -u admin@example.com:password123 \
  -XPOST http://localhost:5080/api/default/default/_json \
  -d '[{"message": "test log entry", "source": "curl"}]'
```

If successful, you should receive a response like:
```json
{"code": 200,"status": [{"name": "default","successful": 1,"failed": 0}]}
```

## Cleanup

```bash
kubectl delete -f flux-delivery.yaml -n openobserve
kubectl delete -f namespace.yaml
```
