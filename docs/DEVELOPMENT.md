# Copilot Instructions for ORISE Infrastructure Examples

## Project Overview
- Kubernetes configuration examples for deploying services with Flux GitOps.
- Contains Kustomize overlays and Flux Helm Release configurations.
- Designed for multi-environment deployment: air-gapped-edge, on-premises-edge, private-cloud, public-cloud.
- All changes via pull requests, reviewed, and merged to `main`.

## Repository Structure

The repository supports two structural patterns for examples:

### 1. Simple Structure (e.g., `openobserve`, `longhorn`)
Used for services with a single deployment configuration.
- **`<service-name>/`**
  - **`source.yaml`**: Flux HelmRepository or GitRepository.
  - **`release.yaml`**: Flux HelmRelease.
  - **`kustomization.yaml`**: Entry point, aggregates resources.
  - **`namespace.yaml`**: Namespace definition.
  - **`README.md`**: Instructions.

### 2. Multi-Profile Structure (e.g., `postgres`, `nginx`)
Used when a service has distinct architectural patterns (e.g., Edge vs. Private Cloud).
- **`<service-name>/`**
  - **`README.md`**: Main documentation covering all profiles.
  - **`base/`** (Optional): Shared resources (Deployment, Service, ConfigMaps).
  - **`<profile-name>/`** (e.g., `edge`, `private-cloud`): Self-contained Kustomize overlay.
    - **`kustomization.yaml`**: Profile-specific aggregation and patches.
    - **`source.yaml`**, **`release.yaml`**: Profile-specific Flux resources.
    - **`namespace.yaml`**: Profile-specific namespace.

- **`resources/`**: Shared cluster-level configurations (e.g., `kind-config.yaml`).
- **`docs/`**: Centralized documentation and development guidelines.

## Key Concepts

### Kustomize Pattern
- **Namespace Handling:** `namespace.yaml` should be present but **commented out** in `kustomization.yaml` resources list. This separates the GitOps configuration (which assumes the namespace exists) from the manual testing workflow.
  ```yaml
  resources:
    # For testing we keep the namespace manually, on production the repo is only one not for branch.
    #- namespace.yaml
    - source.yaml
    - release.yaml
  ```

### Flux Pattern
- **Source** (`source.yaml`) — Points to upstream Helm charts or Git repositories.
- **Release** (`release.yaml`) — Declares Helm chart versions and values overrides.
- **Production Alignment:**
  - **Dev Examples:** Self-contained. Flux resources live in the app namespace.
  - **Production Examples (e.g., Nginx):** Flux resources live in `flux-system` and target the app namespace.

### Sealed Secrets / Encryption
- Sensitive data is encrypted with Age.
- `age.agekey` is the local decryption key — **never commit to Git**.

## Development Workflow

### 1. **Add a New Service Example**
   - Determine if it needs profiles (`edge`/`private-cloud`) or a simple structure.
   - Create the necessary folders and manifests.
   - Use `resources/kind-config.yaml` for local testing (ensures Ingress/Storage compatibility).

### 2. **Test Locally ("Happy Path")**
   - Manually create the target namespace: `kubectl create namespace <name>`.
   - Deploy using Flux CLI:
     ```bash
     flux create source git ...
     flux create kustomization ...
     ```
   - Documentation should be minimal and focus on these success steps.

### 3. **Naming Conventions**
   - **Folders**: lowercase, kebab-case.
   - **Files** (in the deployment directory):
     - `source.yaml` (formerly `repository.yaml`)
     - `release.yaml`
     - `namespace.yaml`
     - `kustomization.yaml`

## Best Practices

1. **Self-Contained Examples**: Do not assume `flux-system` existence unless explicitly demonstrating a production pattern.
2. **Immutability**: Use explicit versions for container images and Helm charts.
3. **Shared Infrastructure**: Use `resources/kind-config.yaml` to guarantee environment consistency (HAProxy ports, Longhorn mounts).