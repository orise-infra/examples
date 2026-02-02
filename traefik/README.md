# Traefik Ingress Controller (OSS)

Example of deploying Traefik v3 using Flux CD.

## Patterns Demonstrated
- Ingress Controller deployment via Helm OCI/Repository.
- High Availability (Private Cloud) vs Single Node (Edge) profiles.
- Automatic namespace creation.

## Deployment

### 1. Create Source
```bash
flux create source git traefik-example \
  --url=https://github.com/orise-infra/examples \
  --branch=main \
  --path=./traefik
```

### 2. Verify
```bash
kubectl get pods -n traefik
```

