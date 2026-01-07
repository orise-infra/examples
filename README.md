# ORISE Infrastructure Examples

Kubernetes configuration examples for deploying services with Flux.

## Usage

These examples are designed to be deployed using Flux GitOps. Flux will automatically apply and reconcile these Kustomize configurations from your Git repository.

### Manual Testing

To test configurations locally before committing:

```bash
kubectl apply -k <folder-name>/
```

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

