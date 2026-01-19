# Copilot Instructions for ORISE Infrastructure Examples

## Project Overview
- Kubernetes configuration examples for deploying services with Flux GitOps.
- Contains Kustomize overlays and Flux Helm Release configurations for multiple platform services.
- Designed for multi-environment deployment: air-gapped-edge, on-premises-edge, private-cloud, public-cloud.
- All changes via pull requests, reviewed, and merged to `main`.

## Repository Structure

```
examples/
├── nginx/                          # Nginx ingress example
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── openobserve/                    # OpenObserve observability stack
│   ├── namespace.yaml
│   ├── source.yaml                 # Flux HelmRepository source
│   ├── release.yaml                # Flux HelmRelease
│   ├── secret.yaml                 # Sealed secrets or encrypted credentials
│   ├── flux-delivery.yaml          # Flux delivery pipeline config
│   └── kustomization.yaml
├── postgresql/                     # PostgreSQL Operator + Instance
│   ├── namespace/                  # Namespace setup
│   ├── operator/                   # PostgreSQL Operator deployment
│   ├── instance/                   # PostgreSQL cluster instance
│   ├── resources/                  # Supporting resources
│   ├── tests/                      # Connectivity and smoke tests
│   ├── age.agekey                  # Age encryption key (DO NOT COMMIT)
│   └── kustomization.yaml
└── README.md
```

## Key Concepts

### Kustomize Pattern
- Each service folder contains a `kustomization.yaml` that aggregates Kubernetes manifests.
- Use `kustomization.yaml` to:
  - Define resource order (namespaces first, operators before instances)
  - Apply patches and overlays for different deployment models
  - Reference external bases and components

### Flux Pattern
- **HelmRepository** (`source.yaml`) — Points to upstream Helm chart repositories.
- **HelmRelease** (`release.yaml`) — Declares Helm chart version, values overrides, and upgrade strategy.
- **Kustomization** (`flux-delivery.yaml`) — Flux source reference and reconciliation settings.

### Sealed Secrets / Encryption
- Sensitive data (credentials, tokens) are encrypted with Age or Sealed Secrets.
- `age.agekey` is the local decryption key — **never commit to Git**.
- Use `.gitignore` to exclude encryption keys.

### Testing
- `tests/` folders contain smoke tests: connectivity checks, resource health validation.
- Run tests locally: `kubectl apply -f tests/`

## Development Workflow

### 1. **Add a New Service Example**
   - Create a new folder: `examples/<service-name>/`
   - Add namespace, source (HelmRepository), release (HelmRelease), and kustomization:
     ```yaml
     ---
     apiVersion: v1
     kind: Namespace
     metadata:
       name: <service-name>
     ---
     apiVersion: source.toolkit.fluxcd.io/v1
     kind: HelmRepository
     metadata:
       name: <chart-repo>
       namespace: <service-name>
     spec:
       interval: 1h
       url: https://charts.example.com
     ---
     apiVersion: helm.toolkit.fluxcd.io/v2
     kind: HelmRelease
     metadata:
       name: <service-name>
       namespace: <service-name>
     spec:
       chart:
         spec:
           chart: <chart-name>
           version: "X.Y.Z"
           sourceRef:
             kind: HelmRepository
             name: <chart-repo>
       values:
         # Service-specific values
     ---
     apiVersion: kustomize.config.k8s.io/v1beta1
     kind: Kustomization
     metadata:
       name: <service-name>
     resources:
       - namespace.yaml
       - source.yaml
       - release.yaml
     ```

### 2. **Test Locally**
   - Dry-run with Kustomize:
     ```bash
     kubectl kustomize <service-folder>/
     ```
   - Apply to a local cluster:
     ```bash
     kubectl apply -k <service-folder>/
     ```
   - Check reconciliation status:
     ```bash
     kubectl get helmrelease -n <namespace>
     flux get all
     ```

### 3. **Add Encrypted Secrets**
   - For PostgreSQL example, age encryption is used:
     ```bash
     # Encrypt a secret
     age -R <age-key> -o secret.yaml.age secret.yaml
     
     # Decrypt (local development only)
     age -d -i age.agekey secret.yaml.age > secret.yaml
     ```
   - Reference in `kustomization.yaml` via `sops` or Sealed Secrets controller.

### 4. **Add Tests**
   - Create a `tests/` folder with connectivity and health checks:
     ```yaml
     apiVersion: batch/v1
     kind: Job
     metadata:
       name: service-connectivity-test
     spec:
       template:
         spec:
           containers:
           - name: test
             image: alpine:latest
             command: ["sh", "-c", "wget -O- http://service:port/health"]
           restartPolicy: Never
     ```

### 5. **Validate Manifests**
   - Use `kubeval` to validate YAML structure:
     ```bash
     kubeval <service-folder>/*.yaml
     ```
   - Check Flux sources and releases:
     ```bash
     flux build kustomization <kustomization-name>
     ```

## Naming Conventions

- **Folders**: `<service-name>/` (lowercase, kebab-case if multi-word)
- **Files**: 
  - `namespace.yaml` — Namespace definition
  - `source.yaml` — HelmRepository or GitRepository
  - `release.yaml` — HelmRelease
  - `secret.yaml` — Sealed/encrypted secrets
  - `flux-delivery.yaml` — Flux Kustomization/HelmRelease reconciliation config
  - `kustomization.yaml` — Kustomize overlay
  - `*-job.yaml` — Test jobs

## Best Practices

### Configuration
1. **Immutability**: Use explicit chart versions, no `latest` tags.
2. **Namespace isolation**: Each service in its own namespace; use network policies.
3. **Resource requests/limits**: Define CPU and memory limits for all containers.
4. **Health checks**: Include `livenessProbe` and `readinessProbe`.

### Secrets Management
1. **Never commit unencrypted secrets** — use Age, Sealed Secrets, or external vaults.
2. **Rotate encryption keys** annually; store age.agekey in secure credential vault.
3. **Use separate secrets per environment** (dev, staging, prod).

### Deployment Safety
1. **Pre-deployment validation**: Run `kubeval`, `kustomize build`, and `flux build`.
2. **Smoke tests**: Include connectivity and health checks in `tests/`.
3. **Progressive delivery**: Use Flux `interval` and `suspend` fields for rollout control.
4. **Evidence collection**: Log all deployments and reconciliation events.

## Common Tasks

### Deploy PostgreSQL Stack
```bash
kubectl apply -k postgresql/
kubectl wait --for=condition=ready pod -l app=postgresql -n postgresql --timeout=5m
kubectl exec -it pod/<pg-pod> -n postgresql -- psql -U postgres -d postgres
```

### Monitor Flux Reconciliation
```bash
flux get all --all-namespaces
flux logs --follow --all-namespaces
```

### Create Git Bundle for Air-Gapped Deploy
```bash
git bundle create examples-all.bundle --all
# Transfer bundle to air-gapped environment
git clone examples-all.bundle examples-repo
```

## References

- **Kustomize Documentation**: https://kustomize.io
- **Flux CD**: https://fluxcd.io
- **Kubernetes Operators**: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
- **Age Encryption**: https://age-encryption.org
- **Sealed Secrets**: https://github.com/bitnami-labs/sealed-secrets
- **PostgreSQL Operator**: https://github.com/zalando/postgres-operator

## Contribution Guidelines

See [CONTRIBUTING.md](../../infra-docs/CONTRIBUTING.md) for general workflow.

**Additional rules for examples:**
- All manifests must pass `kubeval` validation.
- Include namespace definitions and resource isolation.
- Document any external dependencies (e.g., CRDs, operators).
- Test locally before submitting PR.
- Use relative paths in Kustomize `bases` and `resources` fields.
