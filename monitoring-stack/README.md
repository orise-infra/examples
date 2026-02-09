# Monitoring Stack Example - Multi-Environment

This example demonstrates a monitoring stack consisting of InfluxDB 1.8, Chronograf, and Fluent-bit, using **Kustomize overlays** for multi-environment deployments.

## Architecture

### Components

- **InfluxDB 1.8**: Time-series database for storing logs and metrics
- **Chronograf**: Visualization tool for InfluxDB data
- **Fluent-bit**: Log processor and forwarder, configured to send logs to InfluxDB


## Environment Differences

| Component               | Edge        | Private Cloud          |
|-------------------------|-------------|------------------------|
| **InfluxDB Storage**    | 1Gi         | 100Gi (fast-ssd)       |
| **InfluxDB Memory**     | 256Mi-512Mi | 4Gi-8Gi                |
| **InfluxDB CPU**        | 100m-500m   | 2-4                    |
| **Chronograf Replicas** | 1           | 3                      |
| **Chronograf Ingress**  | No          | Yes (TLS)              |
| **Fluent-bit Memory**   | 64Mi-128Mi  | 256Mi-512Mi            |
| **Authentication**      | Disabled    | Enabled                |
| **Monitoring**          | No          | Prometheus annotations |

## Deployment

This example follows the [Canonical Namespace Handling Strategy](../README.md#canonical-namespace-handling-strategy).

### Prerequisites

```bash
# Install Flux CLI if not already installed
brew install fluxcd/tap/flux

# Verify cluster access
kubectl cluster-info
```

### Deploy to Edge

```bash
export ENVIRONMENT="edge"

# Build and preview manifests
kubectl kustomize overlays/$ENVIRONMENT

# Apply directly
kubectl apply -k overlays/$ENVIRONMENT

# Or use Flux (GitOps approach)
flux create source git monitoring-stack \
  --url=https://github.com/orise-infra/examples \
  --branch=main \
  --namespace=flux-system

flux create kustomization monitoring-stack-$ENVIRONMENT \
  --source=GitRepository/monitoring-stack \
  --path="./monitoring-stack/overlays/$ENVIRONMENT" \
  --prune=true \
  --namespace=flux-system
```

### Deploy to Private Cloud

```bash
export ENVIRONMENT="private-cloud"

# IMPORTANT: Update production secrets before deployment
# Create secret for InfluxDB admin password
kubectl create secret generic influxdb-auth \
  --from-literal=admin-password='YOUR-SECURE-PASSWORD' \
  --namespace=monitoring-stack

# Apply configuration
kubectl apply -k overlays/$ENVIRONMENT

# Or with Flux (recommended for private cloud)
flux create kustomization monitoring-stack-$ENVIRONMENT \
  --source=GitRepository/monitoring-stack \
  --path="./monitoring-stack/overlays/$ENVIRONMENT" \
  --prune=true \
  --namespace=flux-system
```

## Configuration Management

### Adding New Values

To add or modify Helm values for a specific environment:

1. Edit the appropriate `values-<component>.yaml` file in the overlay directory
2. Commit and push changes
3. Flux will automatically reconcile (if using GitOps)

Example: Increase InfluxDB storage in private cloud:

```bash
# Edit overlays/private-cloud/values-influxdb.yaml
persistence:
  size: 200Gi  # Changed from 100Gi
```

### Environment-Specific Labels

Each environment automatically gets labeled:
- **Edge**: `orise.io/environment=edge`
- **Private Cloud**: `orise.io/environment=private-cloud`, `orise.io/criticality=high`

These labels enable:
- Resource filtering and monitoring
- Network policies per environment
- Cost allocation tracking

## Verification

```bash
# Check the status
kubectl get all -n monitoring-stack

# Check Flux reconciliation (if using GitOps)
flux get kustomizations monitoring-stack-$ENVIRONMENT

# Check HelmRelease status
flux get helmreleases -n monitoring-stack

# Check ConfigMaps (generated from values files)
kubectl get configmaps -n monitoring-stack

# View logs
kubectl logs -n monitoring-stack -l app.kubernetes.io/name=influxdb
kubectl logs -n monitoring-stack -l app.kubernetes.io/name=chronograf
kubectl logs -n monitoring-stack -l app.kubernetes.io/name=fluent-bit
```

## Testing Values

Preview what will be applied before deployment:

```bash
# Preview edge environment
kubectl kustomize overlays/edge

# Preview private cloud environment
kubectl kustomize overlays/private-cloud

# Compare differences between environments
diff <(kubectl kustomize overlays/edge) <(kubectl kustomize overlays/private-cloud)
```

## Rollback

### Using Flux (GitOps)

```bash
# Revert git commit
git revert <commit-hash>
git push origin main

# Or force sync to previous revision
flux reconcile kustomization monitoring-stack-$ENVIRONMENT --with-source
```

### Manual Rollback

```bash
# Get previous HelmRelease revision
flux get helmreleases -n monitoring-stack

# Rollback specific release
helm rollback influxdb <revision> -n monitoring-stack
```

## Cleanup

```bash
# Remove specific environment
kubectl delete -k overlays/$ENVIRONMENT

# Or with Flux
flux delete kustomization monitoring-stack-$ENVIRONMENT

# Remove namespace (removes everything)
kubectl delete namespace monitoring-stack
```

## Private Cloud Considerations

### Security

1. **Secrets Management**: Replace plaintext passwords with Sealed Secrets or external secret stores
   ```bash
   # Use sealed-secrets or external-secrets operator
   kubectl create secret generic influxdb-auth \
     --from-literal=admin-password='...' \
     --dry-run=client -o yaml | \
     kubeseal -o yaml > overlays/private-cloud/influxdb-sealed-secret.yaml
   ```

2. **Network Policies**: Restrict traffic between components
   ```bash
   # Add network policies to overlays/private-cloud/
   ```

3. **RBAC**: Implement least-privilege access

### High Availability

- Private cloud uses 3 Chronograf replicas
- Consider StatefulSet for InfluxDB clustering
- Use pod anti-affinity for distribution

### Monitoring

Private cloud includes Prometheus annotations:
```yaml
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8086"
```

### Backup & Recovery

Implement InfluxDB backup strategy:
```bash
# Backup script (add to CronJob)
kubectl exec -n monitoring-stack influxdb-0 -- \
  influxd backup -portable /backup/$(date +%Y%m%d)
```

## Migration from Old Structure

If migrating from the old flat structure:

```bash
# Old structure had everything in root with inline values
# New structure uses base + overlays with separate values files

# Choose your target environment and deploy
kubectl apply -k overlays/edge
```

## See Also

- [Kustomize Documentation](https://kustomize.io/)
- [Flux Multi-Environment Setup](https://fluxcd.io/docs/guides/repository-structure/)
- [InfluxDB Helm Chart](https://github.com/influxdata/helm-charts)

