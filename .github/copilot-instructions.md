# Copilot Instructions for ORISE Infrastructure Examples

## Project Overview
- Reference Kubernetes configurations demonstrating Flux CD GitOps deployment patterns.
- **Context:** Part of Orise Infrastructure ecosystem; provides production-ready examples for product teams deploying to edge, on-prem, and cloud environments.
- **Goal:** Enable product teams to quickly adopt GitOps patterns with working examples they can fork and adapt.
- All examples use Kustomize and are designed for Flux CD reconciliation.
- Repository: https://github.com/orise-infra/examples

## Architecture Context
**Deployment Pattern:**
- Examples assume pre-installed Flux CD (tenant model, not bootstrap).
- Namespace isolation: Each example is self-contained within its own namespace.
- GitOps-driven: Flux reconciles from Git; no manual kubectl apply in production.
- Read-only Git sources: Uses public HTTP URLs to avoid authentication complexity in examples.

**Target Platforms:**
- StratumOS edge devices (air-gapped, K3s)
- On-premises Kubernetes clusters
- Private cloud Kubernetes clusters
- Public cloud managed Kubernetes (with guardrails)

All examples must work across these deployment models with minimal variation.

## Repository Structure
```
examples/
├── README.md                    # Quick start and bundle creation instructions
├── <service-name>/             # One folder per example service
│   ├── README.md               # Service-specific deployment guide
│   ├── kustomization.yaml      # Kustomize base configuration
│   ├── namespace.yaml          # Namespace definition (if needed)
│   ├── flux-delivery.yaml      # Flux GitRepository + Kustomization
│   └── *.yaml                  # K8s resource manifests
```

**Current examples:**
- `postgres/` — Stateful PostgreSQL cluster via CloudNativePG
- `nginx/` — Basic stateless web server deployment
- `traefik/` — Ingress Controller (OSS) via Helm OCI
- `openobserve/` — Helm-based observability stack with Flux HelmRelease

## Content Conventions
**Every example folder MUST include:**
1. `README.md` with sections:
   - Overview (what the example demonstrates)
   - Quick Start (step-by-step deployment)
   - Verification (how to confirm it works)

2. `kustomization.yaml` following best practices:
   - Explicit namespace declaration
   - Common labels (app.kubernetes.io/name)
   - Resources list

**Additional rules:**
- One example = one purpose; keep it simple and focused.
- Minimal documentation; YAML goes in folders.

## Example Patterns

### Stateful Example (postgres)
- Uses CloudNativePG operator
- Demonstrates storage integration (Longhorn)
- HA (Private Cloud) vs Single-node (Edge) profiles

### Stateless Example (nginx)
- Simple Deployment + Service
- Demonstrates basic Kustomize patterns

### Ingress Example (traefik)
- Traefik v3 OSS Helm Chart
- Demonstrates traffic routing and HA patterns

### Git Bundle Workflow
Examples repo supports air-gapped distribution via git bundles:
```bash
# Create bundle of current branch
git bundle create examples-head.bundle HEAD

# Create bundle of all branches
git bundle create examples-all.bundle --all

# Verify bundle
git bundle verify examples-all.bundle
```

Document this in READMEs when relevant for air-gapped edge deployments.

## Development Workflow
1. Create new example folder: `<service-name>/`
2. Add manifests following structure above
3. Test locally: `kubectl apply -k <service-name>/`
4. Test with Flux: Apply flux-delivery.yaml to a test cluster
5. Verify reconciliation: `flux get kustomization -n <namespace>`
6. Document in service README.md
7. Update root README.md with link to new example

## Integration with Infra Ecosystem
- **infra-docs**: Architecture documentation and ADRs live at https://orise-infra.github.io/infra-docs
- **infra-portal**: User-facing runbooks and how-to guides reference these examples
- **Responsibility model**: Infrastructure provides examples; product teams own their forks and configurations

## Key Principles
- **Simplicity**: Examples should be minimal but production-ready
- **Portability**: Must work across air-gapped edge, on-prem, and cloud
- **GitOps-first**: Always show Flux reconciliation, not imperative kubectl
- **Namespace isolation**: Tenant model, not admin bootstrap
- **Documentation**: Every example is a teaching tool; explain the pattern

## References
**Internal:**
- Infrastructure docs: https://orise-infra.github.io/infra-docs
- Team charter: https://orise-infra.github.io/infra-docs/01-team/
- Flux runbooks: See infra-portal/docs/runbooks/

**External:**
- Flux CD: https://fluxcd.io/docs/
- Kustomize: https://kubectl.docs.kubernetes.io/references/kustomize/
- Kubernetes: https://kubernetes.io/docs/
