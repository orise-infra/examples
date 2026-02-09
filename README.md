# ORISE Infrastructure Examples

Kubernetes configuration examples for deploying services with Flux.

## Usage

These examples are designed to be deployed using Flux.

For detailed instructions and deployment steps, see [docs/setup.md](docs/setup.md).

## Canonical Namespace Handling Strategy

We follow a strict pattern for namespace management to ensure consistency and avoid configuration drift. This is the **production-aligned pattern (Pattern 2)**.

### Key Principles

1. **Application Namespace**: Each application has its own namespace defined in a `namespace.yaml` file (e.g., `nginx-edge`, `postgres-db`).
   - The namespace is created as part of the application deployment.
   - All application resources (Pods, Services, ConfigMaps, etc.) live in this namespace.

2. **Single Source of Truth for Namespace**: The `namespace.yaml` file is the **single source of truth**.
   - It is **included as a resource** in the `kustomization.yaml` resources list.
   - The `kustomization.yaml` file must **NOT** have a top-level `namespace:` field (this avoids duplication and brittleness).
   - Base folders do not need `namespace.yaml`.
   - Example:
     ```yaml
     resources:
       - namespace.yaml
       - source.yaml
       - release.yaml
     ```

3. **Flux Resources Location**: 
   - **Flux system resources** (`GitRepository`, `Kustomization`) are created in the **`flux-system` namespace**.
   - **Application resources** (Helm releases, Ingress, etc.) are deployed to their own **application namespace**.
   - This separation provides security isolation and aligns with production practices.

### Quick Overview

Each example folder contains a `README.md` with specific deployment instructions. The deployment follows the pattern defined above:
- Flux resources go in `flux-system` namespace
- Application resources go in their own namespace
- Namespaces are defined once in `namespace.yaml` (not duplicated in `kustomization.yaml`)

## Available Examples

- **nginx**: Nginx deployment examples for Edge and Private Cloud.
- **longhorn**: Longhorn distributed block storage.
- **postgres**: PostgreSQL examples (Edge: StatefulSet, Private Cloud: Operator).
- **openobserve**: Observability platform.
- **metrics-server**: Cluster-wide aggregator of resource usage data.

## Create Git Bundle

To create a git bundle of the current branch (HEAD):

```bash
git bundle create examples-head.bundle HEAD
```

To bundle all branches and tags:

```bash
git bundle create examples-all.bundle --all
```

## Verify Git Bundle

To verify a git bundle is valid:

```bash
git bundle verify examples-all.bundle
```

This checks that the bundle is complete and can be used to clone or fetch from.
