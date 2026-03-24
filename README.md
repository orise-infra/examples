# ORISE Infrastructure Examples

Kubernetes configuration examples for deploying services with Flux.

## Structure & Environments

This repository is strictly organized into two target environments, demonstrating how to deploy workloads with varying requirements:

1.  **`/edge`**: Demonstrates the stateless deployment pattern.
    - Lightweight, single-node configurations.
    - Uses standard Kubernetes manifests without Helm.
    - Currently uses Nginx as the example workload.

2.  **`/private-cloud`**: Demonstrates the stateful deployment pattern.
    - High-Availability (HA), multi-node configurations.
    - Uses Kubernetes Operators (CloudNativePG) and distributed storage (Longhorn).
    - Currently uses PostgreSQL as the example workload.

*Note: Mid-term, this repository will evolve to deploy a single unified application that simply adapts to the target environment with Stateless and Stateful Loads.

## Deployment Setup

These examples are designed to be deployed using Flux CD.

For detailed instructions and deployment steps, see [docs/setup.md](docs/setup.md).

## Canonical Namespace Handling Strategy

We follow a strict pattern for namespace management to ensure consistency and avoid configuration drift. This is the **production-aligned pattern (Pattern 2)**.

### Key Principles

1. **Application Namespace**: Each application has its own namespace defined in a `namespace.yaml` file.
   - The namespace is created as part of the application deployment.
   - All application resources (Pods, Services, ConfigMaps, etc.) live in this namespace.

2. **Single Source of Truth for Namespace**: The `namespace.yaml` file is the **single source of truth**.
   - It is **included as a resource** in the `kustomization.yaml` resources list.
   - The `kustomization.yaml` file must **NOT** have a top-level `namespace:` field (this avoids duplication and brittleness).

3. **Flux Resources Location**: 
   - **Flux system resources** (`GitRepository`, `Kustomization`) are created in the **`flux-system` namespace**.
   - **Application resources** (Helm releases, Ingress, etc.) are deployed to their own **application namespace**.
   - This separation provides security isolation and aligns with production practices.

## Working with Git Bundles

To create a git bundle of the current branch (HEAD):

```bash
git bundle create examples-head.bundle HEAD
```

To bundle all branches and tags:

```bash
git bundle create examples-all.bundle --all
```

To verify a git bundle is valid:

```bash
git bundle verify examples-all.bundle
```
