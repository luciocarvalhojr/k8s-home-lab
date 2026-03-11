# k8s-home-lab

This repository contains the configurations to bootstrap a complete Kubernetes home lab using a GitOps-centric approach with ArgoCD.

## Architecture

```
                          ┌─────────────────────────────────────────┐
                          │            Internet / User               │
                          └────────────────┬────────────────────────┘
                                           │ HTTPS *.capihome.xyz
                                           ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  Home Network / Bare-Metal Cluster (k3s)                                     │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │  MetalLB  (metallb-system)                                          │     │
│  │  LoadBalancer IP pool: 192.168.0.161–192.168.0.165                  │     │
│  └────────────────────────────┬────────────────────────────────────────┘     │
│                               │                                              │
│                               ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │  Traefik  (kube-system)   — Ingress Controller                      │     │
│  │  Terminates TLS · Routes traffic by hostname                        │     │
│  └──┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┘     │
│     │          │          │          │          │          │                  │
│     ▼          ▼          ▼          ▼          ▼          ▼                  │
│  argocd    grafana   auth(IDP)  headlamp  obs-auth   myapp / go-api          │
│  .capiho…  .capiho…  .capiho…  .capiho…  .capiho…   .capiho…                │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  GitOps & Secret Management                                                  │
│                                                                              │
│  ┌──────────────────────────┐     ┌──────────────────────────────────┐       │
│  │  ArgoCD  (argocd)        │────▶│  GitHub: k8s-home-lab (main)     │       │
│  │  argocd.capihome.xyz     │     │  App-of-Apps pattern             │       │
│  │  OIDC via Authentik      │     └──────────────────────────────────┘       │
│  └────────────┬─────────────┘                                                │
│               │ manages all apps below                                       │
│               ▼                                                              │
│  ┌──────────────────────────┐                                                │
│  │  Sealed Secrets          │  ← Controller decrypts SealedSecrets in git   │
│  │  (sealed-secrets ns)     │    and creates real K8s Secrets in cluster     │
│  └──────────────────────────┘                                                │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  DNS & TLS                                                                   │
│                                                                              │
│  ┌──────────────────────────┐     ┌──────────────────────────────────┐       │
│  │  ExternalDNS (kube-sys)  │────▶│  Cloudflare DNS  (capihome.xyz)  │       │
│  │  Watches Ingress/Service │     └──────────────────────────────────┘       │
│  │  → creates DNS records   │                    ▲                           │
│  └──────────────────────────┘                    │ DNS-01 challenge          │
│  ┌──────────────────────────┐                    │                           │
│  │  cert-manager (cert-mgr) │────────────────────┘                          │
│  │  Let's Encrypt via       │  wildcard cert: *.capihome.xyz                │
│  │  Cloudflare DNS-01       │                                                │
│  └──────────────────────────┘                                                │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Identity Provider                                                           │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐      │
│  │  Authentik  (idp)           auth.capihome.xyz                      │      │
│  │  SSO / OIDC for: ArgoCD · Grafana · Headlamp · Observatory · k8s  │      │
│  │  PostgreSQL + Redis (internal)                                     │      │
│  └────────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Monitoring                                                                  │
│                                                                              │
│  ┌──────────────────────────────────────────────────────┐                    │
│  │  kube-prometheus-stack  (monitoring)                 │                    │
│  │  Prometheus  — scrapes all cluster metrics           │                    │
│  │  Grafana     — grafana.capihome.xyz  (OIDC login)    │                    │
│  │  Alertmanager                                        │                    │
│  └──────────────────────────────────────────────────────┘                    │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Storage                                                                     │
│                                                                              │
│  ┌──────────────────────────────────────────────────────┐                    │
│  │  NFS CSI Driver  (kube-system)                       │                    │
│  │  StorageClass: nfs-rwx  (ReadWriteMany)              │◀── Authentik PG   │
│  └──────────────────────────────────────────────────────┘                    │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Applications                                                                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐         │
│  │  Observatory  (observatory)                                     │         │
│  │                                                                 │         │
│  │  auth-svc  observatory-auth.capihome.xyz                        │         │
│  │  ├── OIDC auth via Authentik                                    │         │
│  │  ├── Issues JWT tokens                                          │         │
│  │  └── Redis  (internal cache)                                    │         │
│  └─────────────────────────────────────────────────────────────────┘         │
│                                                                              │
│  ┌─────────────────────┐  ┌───────────────────────────┐                     │
│  │  Headlamp (default) │  │  go-api  (go-api)          │                     │
│  │  my-headlamp.capi…  │  │  Helm chart from own repo  │                     │
│  │  K8s UI (OIDC)      │  └───────────────────────────┘                     │
│  └─────────────────────┘                                                     │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐         │
│  │  myapp  (default)  — myapp.capihome.xyz                         │         │
│  │  Kustomize base/overlays pattern (prod overlay active)          │         │
│  └─────────────────────────────────────────────────────────────────┘         │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│  Kubernetes API Access                                                       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐         │
│  │  k8s-apiserver-oidc  (kube-system)                              │         │
│  │  ClusterRoleBinding → kubectl login via Authentik OIDC          │         │
│  └─────────────────────────────────────────────────────────────────┘         │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Namespace Map

| Namespace | Components |
|-----------|-----------|
| `argocd` | ArgoCD (GitOps controller) |
| `sealed-secrets` | Sealed Secrets controller |
| `cert-manager` | cert-manager + ClusterIssuer |
| `kube-system` | Traefik, MetalLB, ExternalDNS, NFS CSI driver, OIDC RBAC |
| `metallb-system` | MetalLB controller + speaker |
| `monitoring` | Prometheus, Grafana, Alertmanager |
| `idp` | Authentik (SSO) + PostgreSQL + Redis |
| `observatory` | auth-svc + Redis |
| `go-api` | go-api |
| `default` | myapp, Headlamp |

## Prerequisites

Before you begin, ensure you have the following installed and configured:

*   A running Kubernetes cluster (e.g., k3s, kubeadm).
*   `kubectl` pointing to your cluster.
*   `helm` (v3+)
*   `kustomize` (v4+)
*   `kubeseal` CLI — for sealing secrets before committing them.
*   `pre-commit` — for local lint, validation, and secret scanning. See [docs/pre-commit.md](docs/pre-commit.md).

```sh
brew install pre-commit
pre-commit install   # run once after cloning
```

## Repository Structure

*   `bootstrap/`: One-time, order-dependent installs — ArgoCD, MetalLB, Sealed Secrets.
*   `infrastructure/`: Platform services — cert-manager, ExternalDNS, monitoring, storage.
*   `identity/`: Auth & access — Authentik IDP, k8s OIDC RBAC.
*   `apps/`: Workloads — observatory, my-headlamp, myapp (base + overlays).
*   `sealed-secrets.pem`: The cluster's public key used to seal secrets with `kubeseal`.

## Deployment Workflow

This setup is designed to be managed by ArgoCD. The recommended deployment process is as follows:

1.  **Clone the Repository:**
    ```sh
    git clone https://github.com/your-username/k8s-home-lab.git
    cd k8s-home-lab
    ```

2.  **Prepare Secrets:**
    Secrets in this repository are encrypted using [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets). The `sealed-secrets-controller` running in the cluster holds the private key and decrypts them automatically. To seal a new secret:
    ```sh
    echo -n "your-secret-value" | kubeseal --raw \
      --name <secret-name> \
      --namespace <namespace> \
      --cert sealed-secrets.pem
    ```
    See each component's `README.md` and its `*-sealed-secret.yaml` / `secrets-sealed.yaml` for details.

3.  **Install ArgoCD:**
    First, you need to install ArgoCD itself into the cluster. This is a manual, one-time step.
    ```sh
    # Follow the instructions in bootstrap/argocd/README.md for the initial ArgoCD installation.
    # Typically, this involves creating the namespace and applying the official Helm chart.
    ```

4.  **Bootstrap the App-of-Apps:**
    Once ArgoCD is running, you apply the root application, which tells ArgoCD to manage all other applications defined in this repository.
    ```sh
    # Ensure you are in the root of the repository
    kubectl apply -f bootstrap/argocd/argodc-applications.yaml
    ```
    ArgoCD will now start deploying all the components listed in `argodc-applications.yaml`. You can monitor the progress from the ArgoCD UI.

## Accessing Your Lab

*   **ArgoCD:** Follow the instructions in `bootstrap/argocd/README.md` to get the initial admin password and access the UI, typically via a port-forward.
*   **Grafana, Headlamp, etc.:** Once deployed by ArgoCD, these applications will be accessible via Ingress. Check the hostnames defined in their respective `ingress.yaml` or `values.yaml` files.

## Components

| Component | Installation Method | Description |
| --- | --- | --- |
| [**ArgoCD**](bootstrap/argocd/) | Helm | Installs and configures ArgoCD for GitOps-style continuous deployment. |
| [**MetalLB**](bootstrap/metallb/) | Helm | Installs MetalLB to provide LoadBalancer services for your bare-metal cluster. |
| [**Sealed Secrets**](bootstrap/sealed-secrets/) | Helm | Installs the Sealed Secrets controller for encrypted secret management. |
| [**cert-manager**](infrastructure/cert-manager/) | Helm | Sets up cert-manager for automated TLS certificate management. |
| [**ExternalDNS**](infrastructure/external-dns/) | Helm | Configures ExternalDNS to automatically manage DNS records for your services and ingresses. |
| [**Monitoring**](infrastructure/monitoring/) | Helm | Sets up the kube-prometheus-stack for a complete cluster monitoring solution with Prometheus and Grafana. |
| [**StorageClass**](infrastructure/storage/) | Direct Apply | Defines a StorageClass for persistent storage using the NFS CSI driver. |
| [**IDP (authentik)**](identity/idp/) | Helm | Deploys authentik as an Identity Provider for centralized authentication. |
| [**k8s-apiserver-oidc**](identity/k8s-apiserver-oidc/) | Direct Apply | Configures the Kubernetes API server to use an OIDC provider for user authentication. |
| [**Observatory**](apps/observatory/) | Helm | Auth service with OIDC and Redis. |
| [**Headlamp**](apps/my-headlamp/) | Helm | Installs Headlamp, a web-based UI for Kubernetes. |
| [**Main Application**](apps/myapp/) | Kustomize | The main application, deployed via ArgoCD. |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
