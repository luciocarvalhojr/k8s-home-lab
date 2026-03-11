# Observatory — Auth Service Secrets Management

This document explains how secrets are managed for `auth-svc` using [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets).

---

## Why Sealed Secrets?

Kubernetes `Secret` resources are only base64-encoded — not encrypted. Committing them to git exposes sensitive values to anyone with repo access.

**Sealed Secrets** solve this by encrypting secrets with a public key. Only the `sealed-secrets-controller` running in the cluster holds the private key and can decrypt them. The encrypted `SealedSecret` resource is safe to commit to git.

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│  Git Repository (k8s-home-lab)                                       │
│                                                                      │
│  observatory/                                                        │
│  ├── auth-svc-values.yaml          ← Helm values (non-secret)        │
│  └── auth-svc-sealed-secret.yaml   ← Encrypted SealedSecret (safe)  │
│                                                                      │
│  argocd/                                                             │
│  ├── obvervatory-auth-svc.yaml     ← ArgoCD app (Helm chart)         │
│  └── observatory-auth-svc-config.yaml  ← ArgoCD app (raw manifests) │
└───────────────────────┬──────────────────────────────────────────────┘
                        │
                        │  ArgoCD syncs both apps
                        ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Kubernetes Cluster (namespace: observatory)                         │
│                                                                      │
│  ┌─────────────────────────────┐                                     │
│  │ SealedSecret: auth-svc-secret│                                    │
│  │  OIDC_CLIENT_SECRET: AgB…   │  ← sealed                          │
│  │  JWT_SECRET: (pending seal) │  ← TODO                            │
│  │  REDIS_URL: AgD…            │  ← sealed                          │
│  └──────────────┬──────────────┘                                     │
│                 │  sealed-secrets-controller decrypts                │
│                 ▼                                                    │
│  ┌─────────────────────────────┐                                     │
│  │ Secret: auth-svc-secret     │  ← real values, only in cluster     │
│  │  OIDC_CLIENT_SECRET: s3cr…  │                                     │
│  │  JWT_SECRET: s3cr…          │                                     │
│  │  REDIS_URL: redis://…       │                                     │
│  └──────────────┬──────────────┘                                     │
│                 │  envFrom.secretRef                                 │
│                 ▼                                                    │
│  ┌─────────────────────────────┐                                     │
│  │ Deployment: auth-svc        │                                     │
│  │  env: OIDC_CLIENT_SECRET ✓  │                                     │
│  │  env: JWT_SECRET ✓          │                                     │
│  │  env: REDIS_URL ✓           │                                     │
│  └─────────────────────────────┘                                     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## How the Two ArgoCD Apps Work Together

| App | File | What it deploys |
|-----|------|-----------------|
| `auth-svc` | `argocd/obvervatory-auth-svc.yaml` | Helm chart — Deployment, Service, Ingress, ConfigMap |
| `observatory-auth-svc-config` | `argocd/observatory-auth-svc-config.yaml` | Raw manifests from `observatory/` — the SealedSecret |

The Helm chart is configured with `externalSecret: true` in `auth-svc-values.yaml`, which tells it **not to create the `auth-svc-secret` Kubernetes Secret**. That Secret is owned entirely by the SealedSecret.

This avoids any conflict between Helm and the SealedSecret controller over the same resource.

```
externalSecret: true  →  Helm skips Secret creation
                      →  SealedSecret controller owns auth-svc-secret
                      →  Deployment finds auth-svc-secret by name (works the same)
```

---

## Secret vs ConfigMap

| Key | Resource | Why |
|-----|----------|-----|
| `PORT`, `ENV`, `JWT_ACCESS_TTL` | ConfigMap (`auth-svc-config`) | Not sensitive |
| `OIDC_ISSUER`, `OIDC_CLIENT_ID` | ConfigMap (`auth-svc-config`) | Not sensitive |
| `OIDC_REDIRECT_URL`, `OTLP_ENDPOINT` | ConfigMap (`auth-svc-config`) | Not sensitive |
| `OIDC_CLIENT_SECRET` | SealedSecret → Secret (`auth-svc-secret`) | Sensitive — OIDC credential |
| `JWT_SECRET` | SealedSecret → Secret (`auth-svc-secret`) | Sensitive — signing key |
| `REDIS_URL` | SealedSecret → Secret (`auth-svc-secret`) | May contain credentials |

---

## How to Seal a New Secret

### Prerequisites

```bash
# kubeseal CLI must be installed
brew install kubeseal

# sealed-secrets.pem is the cluster's public key (committed to repo root)
```

### Seal a value

```bash
echo -n "your-secret-value" | kubeseal \
  --raw \
  --name auth-svc-secret \
  --namespace observatory \
  --cert sealed-secrets.pem
```

The `--name` and `--namespace` flags are important — the encrypted value is **bound** to that specific Secret name and namespace. It cannot be decrypted if used in a different name or namespace.

### Add it to the SealedSecret

Open `observatory/auth-svc-sealed-secret.yaml` and add the key under `spec.encryptedData`:

```yaml
spec:
  encryptedData:
    YOUR_NEW_KEY: <output-from-kubeseal>
```

Commit and push. ArgoCD will sync and the `sealed-secrets-controller` will update the real Secret automatically.

---

## Adding a New Secret Key (End-to-End)

1. **Seal the value** using the command above
2. **Add the encrypted key** to `observatory/auth-svc-sealed-secret.yaml`
3. **If it's a new env var**, update the Helm chart (`helm-charts` repo):
   - Add the key to the Secret template in `templates/service.yaml` (inside the `{{- if not .Values.externalSecret }}` block, for dev use)
   - Add the key with an empty default in `values.yaml` under `secrets:`
   - Bump the chart version
4. **Update `argocd/obvervatory-auth-svc.yaml`** with the new chart version
5. **Commit and push** — ArgoCD handles the rest

---

## File Reference

```
k8s-home-lab/
├── sealed-secrets.pem                          ← cluster public key (seal with this)
├── observatory/
│   ├── auth-svc-values.yaml                    ← Helm values (externalSecret: true set here)
│   └── auth-svc-sealed-secret.yaml             ← SealedSecret (safe to commit)
└── argocd/
    ├── obvervatory-auth-svc.yaml               ← ArgoCD Helm app
    └── observatory-auth-svc-config.yaml        ← ArgoCD raw-manifest app
```
