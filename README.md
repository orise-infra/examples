# ORISE Infrastructure Examples

Reference Kubernetes configurations for deploying services with Flux CD, aligned with the [Orise Infrastructure Architecture](https://orise-infra.github.io/infra-docs/10-architecture/architecture-overview.html).

## Architecture Patterns

These examples demonstrate specific deployment patterns defined in the [Deployment Workflows](https://orise-infra.github.io/infra-docs/10-architecture/deployment-workflows.html). They serve as reference implementations for Product Services (Layer 3) consuming Platform Services (Layer 2).

| Service         | Pattern             | Description                                                                                                                                                     |
|:----------------|:--------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **PostgreSQL**  | **Stateful (CNPG)** | Demonstrates managing stateful workloads using the CloudNativePG operator. Shows dependency management (Operator → Cluster) and storage integration (Longhorn). |
| **NGINX**       | **Stateless (Web)** | Demonstrates a simple stateless web server deployment using standard Kubernetes manifests and Kustomize.                                                        |
| **Traefik**     | **Ingress (OSS)**   | Demonstrates standard Ingress Controller deployment using Traefik v3 OSS Helm charts. Includes profiles for both High Availability and Edge scenarios.          |
| **OpenObserve** | **Helm Delivery**   | Demonstrates deploying third-party applications via Helm Charts wrapped in Flux `HelmRelease`. Shows namespace isolation and secret management.                 |

### Key Principles implied in examples:
1.  **Tenant Model**: Each example is self-contained in a namespace, assuming a pre-existing platform (Layer 1 & 2).
2.  **GitOps-First**: No manual changes; all state is declarative.
3.  **Portability**: Configurations are designed to work across Edge and Cloud models with minimal overlays.

## Usage

These examples are designed to be deployed using **Flux CD**, following the standard [Deployment Workflows](https://orise-infra.github.io/infra-docs/10-architecture/deployment-workflows.html).

### Prerequisites

- **Kubernetes cluster** (Edge or Cloud)
- **Flux CD v2.0+** installed
- **Flux CLI** installed locally

For operational guides and setup instructions, refer to the [Service Runbooks](https://orise-infra.github.io/infra-portal/docs/runbooks/). specifically the [Flux Getting Started guide](https://orise-infra.github.io/infra-portal/docs/runbooks/flux-getting-started.html).

### Deploy a Service

Refer to the specific example's README for deployment instructions:

- **PostgreSQL**: See `postgres/README.md` for CloudNativePG deployment
- **OpenObserve**: See `openobserve/README.md` for observability stack deployment
- **NGINX**: See `nginx/README.md` for Ingress Controller deployment

### Local Testing

To validate configurations locally before deploying:

```bash
# Preview what Kustomize will generate
kubectl kustomize <folder-name>/

# For Flux validation (requires Flux CLI)
flux build kustomization --path <folder-name>/
```

## Air-Gapped / Edge Workflow

For disconnected environments (StratumOS), use Git Bundles as defined in the [Edge Deployment Workflow](https://orise-infra.github.io/infra-docs/10-architecture/deployment-workflows.html#edge-deployment-workflow).

### Create Git Bundle

To create a bundle for transport:

```bash
git bundle create examples-head.bundle HEAD
```

### Verify Git Bundle

To verify a bundle before transport:

```bash
git bundle verify examples-head.bundle
```

This bundle can then be transported to the edge device and consumed by `bundlectl` or Flux.

