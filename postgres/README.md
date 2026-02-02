# PostgreSQL Example (Flux)

## Table of Contents

1.  [Overview](#overview)
2.  [Production Considerations](#production-considerations)
3.  [Prerequisites](#prerequisites)
4.  [Quick Start](#quick-start)
5.  [Cleanup](#cleanup)

## Overview

This example demonstrates deploying PostgreSQL using the **CloudNativePG (CNPG)** operator with **Flux CD** GitOps automation.

*   **Operator**: CloudNativePG manages PostgreSQL cluster lifecycle, backups, and failover.
*   **Storage**: Integrates with **Longhorn** for persistent volumes with configurable replication.
*   **Deployment Models**: Includes profiles for both **private cloud** (3-instance HA cluster) and **edge** (single-instance) deployments.
*   **GitOps**: Flux CD reconciles the desired state from this repository; no manual `kubectl apply` needed in production.

## Production Considerations

*   **Declarative Namespace**: The namespace (`postgres-flux-example`) is defined in the Kustomization and managed by Flux. This is the GitOps-first approach for production.
*   **Operator Dependency**: The operator HelmRelease is applied before Cluster resources, ensuring the CNPG controller is ready to reconcile.
*   **Storage Requirements**: Both profiles assume Longhorn is available. See the [Longhorn Setup Runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/longhorn-kind-setup.html) for local Kind setup instructions.
*   **Production Customization**: Update cluster names, instance counts, storage sizes, and image versions in the appropriate profile (edge or private-cloud) to match your environment.

## Prerequisites

*   Kubernetes Cluster with **Flux CD v2.0+** installed
*   **Longhorn** storage class provisioned on the cluster
*   **Flux CLI** installed locally

See the [Flux Getting Started runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-getting-started.html) for Flux setup instructions.

## Quick Start

### 1. Create GitRepository Source

Use the Flux CLI to create a source pointing to the examples repository:

```bash
flux create source git postgres-example \
  --url=https://github.com/orise-infra/examples \
  --branch=main \
  --interval=1m \
  --namespace=postgres-flux-example
```

For more details on Git sources, see the [Flux documentation](https://fluxcd.io/docs/cmd/flux_create_source_git/).

### 2. Deploy with Flux Kustomization

Create a Kustomization resource to deploy the PostgreSQL example:

```bash
flux create kustomization postgres \
  --source=GitRepository/postgres-example.postgres-flux-example \
  --path=./postgres \
  --prune=true \
  --wait=true \
  --namespace=postgres-flux-example
```

### 3. Verify the Deployment

Wait for the operator to be ready:
```bash
kubectl wait helmrelease/cnpg \
  --namespace=flux-system \
  --for=condition=Ready \
  --timeout=10m
```

Check the PostgreSQL clusters are running:
```bash
# Private cloud cluster
kubectl get postgresql/prod-db -n postgres-flux-example

# Edge cluster
kubectl get postgresql/edge-db -n postgres-flux-example
```

Wait for clusters to be healthy (this may take a few minutes):
```bash
kubectl wait postgresql/prod-db \
  --namespace=postgres-flux-example \
  --for=jsonpath='{.status.currentPrimary}' \
  --timeout=15m
```

Verify all pods are running:
```bash
kubectl get pods -n postgres-flux-example
```

Check Flux reconciliation status:
```bash
flux get kustomization postgres -n postgres-flux-example
flux logs --follow kustomization/postgres -n postgres-flux-example
```

For more monitoring and troubleshooting, see the [Flux Production Operations runbook](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-production-operations.html).

## Cleanup

Remove the PostgreSQL example from your cluster using Flux:

```bash
flux delete kustomization postgres --namespace=postgres-flux-example
flux delete source git postgres-example --namespace=postgres-flux-example
```

To delete the namespace and all remaining resources:

```bash
kubectl delete namespace postgres-flux-example
```

For more details on Flux delete operations, see the [Flux documentation](https://fluxcd.io/docs/cmd/flux_delete/).

