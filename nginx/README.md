# NGINX Web Server (Stateless)

Example of a simple stateless web server deployment.

## Patterns Demonstrated
- Standard Deployment and Service configuration.
- Kustomize-driven labels and namespace management.
- Stateless workload scaling.

## Deployment

```bash
flux create kustomization nginx-web \
  --source=GitRepository/examples \
  --path=./nginx
```