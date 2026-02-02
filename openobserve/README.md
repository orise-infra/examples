# OpenObserve Example (Flux)

This example demonstrates how to deploy OpenObserve using Flux CD with a fully isolated, namespace-scoped configuration.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed.
*   **Flux CLI** installed locally

See the [Flux Getting Started runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-getting-started.html) for Flux setup instructions.

## Deployment Strategy

This example uses a **Tenant** deployment model rather than `flux bootstrap`:
1.  **Pre-installed Flux**: Assumes Flux is already running on the cluster (bootstrapped elsewhere).
2.  **Namespace Isolation**: All resources are confined to the `openobserve` namespace.
3.  **Read-Only Git**: Uses a public HTTP URL for the `GitRepository`, avoiding the need for Git write credentials (PAT) which `flux bootstrap` requires.

## Quick Start

### 1. Create GitRepository Source

Use the Flux CLI to create a source pointing to the examples repository:

```bash
flux create source git openobserve-example \
  --url=https://github.com/orise-infra/examples \
  --branch=main \
  --interval=1m \
  --namespace=openobserve
```

For more details on Git sources, see the [Flux documentation](https://fluxcd.io/docs/cmd/flux_create_source_git/).

### 2. Deploy with Flux Kustomization

Create a Kustomization resource to deploy OpenObserve:

```bash
flux create kustomization openobserve \
  --source=GitRepository/openobserve-example.openobserve \
  --path=./openobserve \
  --prune=true \
  --wait=true \
  --namespace=openobserve
```

### 3. Verify Deployment

Check the Flux reconciliation status and OpenObserve deployment:

```bash
flux get kustomization openobserve -n openobserve
flux get helmrelease -n openobserve
kubectl get pods -n openobserve
```

For more monitoring and troubleshooting, see the [Flux Production Operations runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-production-operations.html).

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

Remove the OpenObserve example using Flux:

```bash
flux delete kustomization openobserve --namespace=openobserve
flux delete source git openobserve-example --namespace=openobserve
```

To delete the namespace and all remaining resources:

```bash
kubectl delete namespace openobserve
```

For more details on Flux delete operations, see the [Flux documentation](https://fluxcd.io/docs/cmd/flux_delete/).

