# Runbook - Vault + External Secrets Operator (PoC)

## Purpose

This runbook describes how to deploy and validate the Vault (dev mode) + External Secrets Operator (ESO) PoC using Flux GitOps, and verify that the sample app consumes the generated Kubernetes Secret.

## Scope

- Cluster: k3s
- GitOps: Flux v2 via Flux Operator
- Secret source: Vault KV v2 (dev mode)
- Sync: External Secrets Operator
- App: busybox Deployment (unchanged)

## Prerequisites

- Flux is installed and reconciling this repo.
- You can run `kubectl` on the control plane or via SSH.
- Vault + ESO manifests are present in Git and referenced by Kustomizations.

---

## 1) Reconcile Flux

Run from control node or via SSH:

```sh
kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A
```

Expected:

- `flux-system` Ready
- `infrastructure` Ready
- `apps` Ready

If needed, force reconcile:

```sh
flux reconcile kustomization infrastructure -n flux-system
flux reconcile kustomization apps -n flux-system
```

---

## 2) Verify Vault and ESO are running

```sh
kubectl -n vault get pods
kubectl -n external-secrets get pods
```

Expected:

- Vault pod(s) Running (dev mode)
- External Secrets Operator pod(s) Running

---

## 3) Seed Vault KV v2 (PoC-only)

Write test secrets into Vault:

```sh
kubectl -n vault exec -it vault-0 -- \
  vault kv put secret/sample-app customer-name="ACME Corp" api-token="abc123"
```

Notes:

- This is the only imperative step in the PoC.
- In production, Vault should be populated by application/pipeline processes, not manual CLI.

---

## 4) Verify SecretStore and ExternalSecret

```sh
kubectl -n sample-app get secretstore
kubectl -n sample-app get externalsecret sample-app-secret
kubectl -n sample-app describe externalsecret sample-app-secret
```

Expected:

- SecretStore: `Ready=True`
- ExternalSecret: `Ready=True` and status `SecretSynced`

---

## 5) Verify Kubernetes Secret is created

```sh
kubectl -n sample-app get secret sample-app-secret -o yaml
```

Expected:

- `data` contains `customer-name` and `api-token`

---

## 6) Verify application consumes secrets

```sh
kubectl -n sample-app get pods
kubectl -n sample-app exec -it deploy/sample-app -- printenv CUSTOMER_NAME
kubectl -n sample-app exec -it deploy/sample-app -- cat /etc/sample-secret/api-token.txt
```

Expected:

- `CUSTOMER_NAME` prints the Vault value
- `/etc/sample-secret/api-token.txt` prints the Vault value

---

## 7) Verify values directly in Vault (optional)

```sh
kubectl -n vault exec -it vault-0 -- vault kv get secret/sample-app
```

---

## Troubleshooting

### ExternalSecret is not Ready

- Check SecretStore status:

  ```sh
  kubectl -n sample-app describe secretstore vault-sample-app
  ```

- Ensure Vault is running:

  ```sh
  kubectl -n vault get pods
  ```

- Confirm Vault has data:

  ```sh
  kubectl -n vault exec -it vault-0 -- vault kv get secret/sample-app
  ```

### App pod stuck in ContainerCreating

- Ensure `sample-app-secret` exists:

  ```sh
  kubectl -n sample-app get secret sample-app-secret
  ```

- Recreate the pod after the Secret appears:

  ```sh
  kubectl -n sample-app delete pod -l app=sample-app
  ```

---

## Notes

- Vault runs in dev mode (in-memory, root token).
- Vault token is stored in Git (encrypt with SOPS for the PoC).
- Production should use Vault Kubernetes auth and TLS, with HA/storage/unseal.
