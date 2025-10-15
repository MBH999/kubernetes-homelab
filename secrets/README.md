
# 🔐 Secrets Management with SOPS & Flux

This repository uses **SOPS** to encrypt Kubernetes secrets and **Flux** to automatically decrypt and apply them to your cluster.

---

## How It Works

- Secrets are stored in the `secrets/` folder, encrypted with SOPS using an **age key**.
- Flux uses a private key stored in the `sops-age` secret (in the `flux-system` namespace) to decrypt secrets when applying manifests.

---

## Initial Setup

1. **Generate an age key:**
   ```bash
   age-keygen -o age.key
   ```

2. **Create the Flux decryption secret:**
   ```bash
   kubectl -n flux-system create secret generic sops-age \
     --from-file=age.agekey=age.key
   ```

3. **Add your public key to `.sops.yaml`:**
   ```yaml
   creation_rules:
     - path_regex: secrets/.*\.sops\.ya?ml
       encrypted_regex: '^(data|stringData)$'
       age:
         - age1xxxxxxx  # your public key
   ```

4. **Configure Flux** to use this key in your `ks-secrets.yaml` (or `ks-infra.yaml`).

---

## Adding or Updating a Secret

1. **Create or edit a file under `secrets/`, e.g.:**


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: azure-creds
  namespace: external-secrets
type: Opaque
stringData:
  ClientID: "<Azure App ID>"
  ClientSecret: "<Secret>"
```


sops --encrypt --in-place secrets/azure-creds.secret.sops.yaml
git add secrets/azure-creds.secret.sops.yaml
git push
age-keygen -o new-age.key
sops --reencrypt --in-place -r secrets/
kubectl -n flux-system create secret generic sops-age \

2. **Encrypt the secret:**
  ```bash
  sops --encrypt --in-place secrets/azure-creds.secret.sops.yaml
  ```

3. **Commit and push the change:**
  ```bash
  git add secrets/azure-creds.secret.sops.yaml
  git commit -m "Update Azure creds"
  git push
  ```

4. **Flux will automatically decrypt and apply the secret.**

---

## Checking Secrets

**Force Flux to sync:**
```bash
flux reconcile kustomization secrets -n flux-system --with-source
```

**Confirm the secret exists:**
```bash
kubectl -n external-secrets get secret azure-creds
```

---

## Rotating the Age Key

1. **Generate a new key:**
  ```bash
  age-keygen -o new-age.key
  ```

2. **Add the new public key to `.sops.yaml`.**

3. **Re-encrypt all secrets:**
  ```bash
  sops --reencrypt --in-place -r secrets/
  ```

4. **Update Flux’s secret:**
  ```bash
  kubectl -n flux-system create secret generic sops-age \
    --from-file=age.agekey=new-age.key --dry-run=client -o yaml | kubectl apply -f -
  ```

---

## Tips

- **Never commit unencrypted secrets.**
- Use `sops <file>` to open and edit an encrypted secret.
- The `sops-age` secret must always exist in `flux-system`.

---

## Quick Reference

| Action                | Command                                                        |
|-----------------------|----------------------------------------------------------------|
| Encrypt new secret    | `sops --encrypt --in-place secrets/<file>.sops.yaml`            |
| Edit existing secret  | `sops secrets/<file>.sops.yaml`                                |
| Re-sync Flux          | `flux reconcile kustomization secrets -n flux-system --with-source` |
| Check secret          | `kubectl -n <namespace> get secret <name>`                     |