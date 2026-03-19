# k8s-home-lab

GitOps-managed bare-metal k3s home lab. Domain: `capihome.xyz`. All infra lives in git; ArgoCD syncs to cluster.

---

## current state

| Component | Version | Notes |
|---|---|---|
| ArgoCD | latest (Helm) | App-of-apps root at `bootstrap/argocd/applications/` |
| Sealed Secrets | 2.18.4 | Secret encryption controller |
| cert-manager | v1.20.0 | Cloudflare DNS-01 |
| Prometheus Stack | 82.10.3 | Grafana + Prometheus + Alertmanager |
| Authentik | 2026.2.0 | Central OIDC/SSO for all services |
| observatory-auth-svc | **1.6.0** (chart 0.1.3) | Merged via PR #32 |
| go-api | 0.1.1 | |
| myapp | prod overlay | Kustomize base/overlays; image traefik/whoami:v1.10.3 |
| MetalLB | v0.15.3 | Load balancer |
| external-dns | v1.20.0 | Cloudflare provider |
| CSI Driver NFS | 4.13.1 | NFS storage provisioner |
| Headlamp | latest (Helm) | Kubernetes dashboard; OIDC via Authentik |

CI: GitHub Actions runs yamllint, kubeconform, kube-linter, gitleaks, checkov, Trivy on every PR and push to main. Pre-commit hooks mirror CI locally.

---

## in progress

Nothing actively in progress. Branch `feat/observatory-auth-svc-update` was merged (PR #32) on main.

---

## known issues

- **JWT_SECRET not sealed**: `apps/observatory/README.md` (line 38) notes this as "pending seal". The `JWT_SECRET` key is commented out in `apps/observatory/auth-svc-sealed-secret.yaml` and has not been added to the SealedSecret yet. Service may not function correctly without it.
- **Plaintext OIDC clientSecret in values files**: `bootstrap/argocd/values.yaml` and `apps/my-headlamp/values.yaml` contain OIDC `clientSecret` in plaintext — noted in their respective READMEs as known issues pending sealing.
- **ArgoCD app file typo**: `bootstrap/argocd/applications/obvervatory-auth-svc.yaml` has misspelling "obvervatory" in the filename (persists from earlier commits).
- **Relaxed security posture**: `.kube-linter.yaml` suppresses CPU/memory limits enforcement, root-container checks, replica count, and network policy requirements — intentional for home-lab but worth tracking.
- **Typo in recent commit messages**: `obvervatory` (misspelling) appears in commits `c085f02`, `fe4f8e1`, `882df1b`.

---

## next steps

1. Seal `JWT_SECRET` and add it to `apps/observatory/auth-svc-sealed-secret.yaml` (see README instructions).
2. Seal the OIDC `clientSecret` in `bootstrap/argocd/values.yaml` and `apps/my-headlamp/values.yaml`.
3. Verify observatory-auth-svc 1.6.0 deploys and OIDC flow works end-to-end.

---

## key decisions

- **GitOps-only**: No `kubectl apply` by hand; everything goes through git → ArgoCD.
- **App-of-apps**: Single root ArgoCD Application manages all child apps.
- **Multi-source Helm**: Charts pulled from external registry (`https://luciocarvalhojr.github.io/helm-charts`); values come from this repo.
- **Sealed Secrets**: Only encryption mechanism for secrets committed to git; SealedSecret controller decrypts at runtime.
- **OIDC everywhere**: Authentik is the single IdP for ArgoCD, Grafana, Headlamp, Kubernetes API, and all apps.
- **Two ArgoCD apps per service** (where secrets are needed): one for the Helm chart, one for the SealedSecret manifests — avoids Helm managing secrets it shouldn't touch (`externalSecret: true`).
- **Layer ordering**: bootstrap → infrastructure → identity → apps. Dependencies are implicit; ArgoCD sync waves not used, so order matters at initial cluster setup.
- **NFS storage**: `192.168.0.110:/mnt/Data/2TB/k3s` via NFS CSI driver for stateful workloads (Authentik PostgreSQL).
- **Cloudflare**: ExternalDNS manages DNS records; cert-manager uses DNS-01 challenges for wildcard TLS certs.
