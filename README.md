# ORISE Infrastructure Examples

Kubernetes configuration examples for deploying services with Flux.

## Usage

These examples are designed to be deployed using Flux. We use a standardized pattern where Flux resources are managed in the `flux-system` namespace.

For detailed instructions on the deployment pattern, see [docs/setup.md](docs/setup.md).

### How to deploy

Each example folder contains a `README.md` with the specific commands needed to deploy it. In general, the process involves:

1. Setting environment variables for the example name and repository.
2. Creating a Flux `GitRepository` source in the `flux-system` namespace.
3. Creating a Flux `Kustomization` in the `flux-system` namespace.

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
