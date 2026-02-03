# Longhorn Distributed Storage

This example demonstrates deploying **Longhorn** as the storage engine for the Kubernetes cluster using Flux CD.

## Components

*   **HelmRepository**: Registers the official Longhorn chart repository.
*   **HelmRelease**: Deploys Longhorn v1.6.x to the `longhorn-system` namespace.
*   **StorageClasses**: Defines custom storage profiles:
    *   `longhorn-ha`: 3 replicas (High Availability) - Recommended for Private Cloud.
    *   `longhorn-edge`: 1 replica (Single Node) - Optimized for Edge (if local-path is not sufficient).

## Prerequisites

*   **ISCSSI**: The nodes must have `open-iscsi` installed (or equivalent for your OS).
*   **Flux CD**: Installed on the cluster.

## Deployment

### 1. Create GitRepository Source

```bash
flux create source git longhorn-example \
  --url=https://github.com/orise-infra/examples \
  --branch=main \
  --interval=1m \
  --namespace=flux-system
```

### 2. Deploy with Flux Kustomization

This creates the `longhorn` Kustomization, which the PostgreSQL example depends on.

```bash
flux create kustomization longhorn \
  --source=GitRepository/longhorn-example.flux-system \
  --path=./longhorn \
  --prune=true \
  --wait=true \
  --health-check-timeout=10m \
  --namespace=flux-system
```

## Verification

Check if the Longhorn pods are running:

```bash
kubectl get pods -n longhorn-system
```

Check if the StorageClasses are created:

```bash
kubectl get sc
```

Access the Longhorn UI (port-forward):

```bash
kubectl port-forward svc/longhorn-frontend -n longhorn-system 8080:80
# Open http://localhost:8080 in your browser
```

