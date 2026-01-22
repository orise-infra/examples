# GitOps Secrets PoC Overview

This repository demonstrates two approaches for managing Kubernetes Secrets in a GitOps workflow:

- **SOPS + age**: Secrets are encrypted in Git and decrypted by Flux at apply time.
- **Vault + External Secrets Operator (ESO)**: Secrets are stored in HashiCorp Vault and synced to Kubernetes using ESO.

## Quick Links

- [SOPS + age Runbook](./runbook-sops-poc.md)
- [Vault + ESO Runbook](./runbook-vault-eso-poc.md)

## Comparison: SOPS vs Vault+ESO

| Aspect             | SOPS + age                     | Vault + External Secrets          |
| ------------------ | ------------------------------ | --------------------------------- |
| Secret storage     | Encrypted in Git               | External Vault server             |
| Runtime dependency | None (decrypted at apply time) | Requires running Vault + ESO      |
| Key management     | age key pair or cloud KMS      | Vault manages keys                |
| Rotation           | Re-encrypt and commit          | Vault handles rotation            |
| Audit              | Git history                    | Vault audit logs                  |
| Complexity         | Simple                         | More infrastructure               |

## Files in This PoC

| File                 | Purpose                                    |
| -------------------- | ------------------------------------------ |
| `secret-plain.yaml`  | Reference plaintext secret (do not commit) |
| `secret-sops.yaml`   | SOPS-encrypted secret (safe to commit)     |
| `deployment.yaml`    | App consuming secret via env and volume    |
| `kustomization.yaml` | Kustomize overlay including the secret     |

## How to Use

- See the runbooks above for step-by-step setup and usage instructions for each approach.
- Each runbook explains how secrets are defined, encrypted, and consumed by the sample app.