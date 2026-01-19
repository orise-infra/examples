# PostgreSQL Example (Flux)

## Table of Contents

1.  [Executive Summary](#executive-summary)
2.  [Production Considerations](#production-considerations)
3.  [Prerequisites](#prerequisites)
4.  [Configuration](#configuration)
5.  [Operator Setup](#operator-setup)
6.  [Instance Setup](#instance-setup)
7.  [Cleanup](#cleanup)

## Executive Summary

This example implements a scalable **PostgreSQL** architecture optimized for read-heavy workloads using the **Zalando Postgres Operator**.

*   **Architecture**: A single cluster split into a **Primary** node for writes and multiple **Replica** nodes for scaling reads.
*   **Management**: **Flux CD** automates the deployment, ensuring the cluster state matches the Git repository.
*   **Security**: **SOPS** is used to encrypt sensitive credentials (secrets), while **ConfigMaps** handle non-sensitive configuration.
*   **Lifecycle**: Leverages Flux features like `dependsOn` to ensure the Operator is ready before the Instance is deployed, and `healthChecks` to verify the cluster status.

## Production Considerations

*   **Namespace Management**: This example demonstrates two approaches:
    *   **Declarative**: The Operator setup (`postgresql/operator/namespaces.yaml`) includes the namespace definitions, allowing Flux to manage them automatically. This is the preferred "GitOps" way for production.
    *   **Imperative**: The Instance setup uses `kubectl create namespace` for demonstration purposes. This reflects environments where namespaces might be pre-provisioned by administrators or managed outside the application's lifecycle.
*   **Secret Security**: The example uses a demonstration-only private key. In a real-world scenario, **never commit private keys to your Git repository**. Use `age` to generate unique keys for your environment. For more details, see `../docs/key-regeneration.md`.

## Prerequisites

*   Kubernetes Cluster with **Flux CD** installed.
*   `age` and `sops` CLI tools installed.

## Configuration

**All commands must be executed from the `postgresql/` directory.**

```bash
cd postgresql
```

Set these environment variables to customize your deployment:

```bash
export GIT_REPO_URL="https://github.com/orise-infra/examples"
export GIT_BRANCH="feat/postgres"
export OPERATOR_NAMESPACE="postgres-operator-system"
export INSTANCE_NAMESPACE="postgres-db"
export OPERATOR_SOURCE_NAME="postgres-operator"
export INSTANCE_SOURCE_NAME="postgres-instance"
export GIT_POLL_INTERVAL="1m"
```

## Operator Setup

### 1. Deploy Operator

Create the `GitRepository` source for the operator:
```bash
flux create source git ${OPERATOR_SOURCE_NAME} \
  --url=${GIT_REPO_URL} \
  --branch=${GIT_BRANCH} \
  --interval=${GIT_POLL_INTERVAL} \
  --namespace=flux-system \
  --export | kubectl apply -f - 
```

Create the `Kustomization` to deploy the operator:
```bash
flux create kustomization ${OPERATOR_SOURCE_NAME} \
  --source=GitRepository/${OPERATOR_SOURCE_NAME}.flux-system \
  --path="./postgresql/operator" \
  --prune=true \
  --wait=true \
  --interval=10m \
  --namespace=flux-system \
  --export | kubectl apply -f - 
```

### 2. Verify Operator

The Operator deployment is asynchronous. Instead of manually polling, use this blocking command to wait until the `HelmRelease` is fully `Ready`:

```bash
kubectl wait helmrelease/${OPERATOR_SOURCE_NAME} \
  --namespace=${OPERATOR_NAMESPACE} \
  --for=condition=Ready \
  --timeout=10m
```

You can optionally inspect the installation events:
```bash
kubectl get events -n ${OPERATOR_NAMESPACE} --sort-by='.lastTimestamp'
```

Finally, confirm the pods are running:
```bash
kubectl get pods -n ${OPERATOR_NAMESPACE}
```

## Instance Setup

### 1. Create Namespace

```bash
# While these namespaces are defined in postgresql/operator/namespaces.yaml,
# we create it here imperatively to demonstrate manual management.
kubectl create namespace ${INSTANCE_NAMESPACE}
```

### 2. Configure Secrets (Demo Only)

This example includes a demonstration-only private key to make the setup process easy.

Create the Kubernetes secret that Flux will use for decryption from the provided demo key:
```bash
kubectl create secret generic sops-age \
  --namespace=${INSTANCE_NAMESPACE} \
  --from-file=age.agekey=../resources/dev-age.agekey
```

### 3. Deploy Instance

Create the `GitRepository` source for the instance:
```bash
flux create source git ${INSTANCE_SOURCE_NAME} \
  --url=${GIT_REPO_URL} \
  --branch=${GIT_BRANCH} \
  --interval=${GIT_POLL_INTERVAL} \
  --namespace=${INSTANCE_NAMESPACE} \
  --export | kubectl apply -f - 
```

Create the `Kustomization` for the instance. This command configures SOPS decryption to use the `sops-age` secret you just created.
```bash
flux create kustomization ${INSTANCE_SOURCE_NAME} \
  --source=GitRepository/${INSTANCE_SOURCE_NAME} \
  --path="./postgresql/instance" \
  --prune=true \
  --wait=true \
  --interval=10m \
  --decryption-provider=sops \
  --decryption-secret=sops-age \
  --namespace=${INSTANCE_NAMESPACE} \
  --target-namespace=${INSTANCE_NAMESPACE} \
  --export | kubectl apply -f - 
```

### 4. Verify Instance

Because the setup is asynchronous, we wait for the Operator to provision the infrastructure and report the status as `Running`:

```bash
kubectl wait postgresql/acid-minimal-cluster \
  --namespace=${INSTANCE_NAMESPACE} \
  --for=jsonpath='{.status.PostgresClusterStatus}'=Running \
  --timeout=5m
```

You can also inspect the events during the provisioning process:

```bash
kubectl get events -n ${INSTANCE_NAMESPACE} --sort-by='.lastTimestamp'
```

Finally, verify the pods are running:
```bash
kubectl get pods -n ${INSTANCE_NAMESPACE}
```

## Cleanup

```bash
kubectl delete namespace ${INSTANCE_NAMESPACE} ${OPERATOR_NAMESPACE}
```