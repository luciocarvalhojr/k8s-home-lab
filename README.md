# k8s-home-lab

This repository contains the configurations to bootstrap a complete Kubernetes home lab using a GitOps-centric approach with ArgoCD.

## Architecture

*TODO: It is recommended to add a diagram here showing how the components interact. For example:*

`User -> Ingress -> App -> Database`

`ExternalDNS -> Manages DNS for Ingress`

`cert-manager -> Provides TLS for Ingress`

`ArgoCD -> Manages all deployments`

`Monitoring -> Scrapes metrics from all components`

## Prerequisites

Before you begin, ensure you have the following installed and configured:

*   A running Kubernetes cluster (e.g., k3s, kubeadm).
*   `kubectl` pointing to your cluster.
*   `helm` (v3+)
*   `kustomize` (v4+)
*   `kubeseal` CLI — for sealing secrets before committing them.

## Repository Structure

*   `argocd/`: Contains the master ArgoCD application that deploys all other applications (apps-of-apps pattern).
*   `base/`: Kustomize base for a sample application.
*   `overlays/`: Kustomize overlays for different environments (e.g., `dev`, `prod`).
*   `cert-manager/`, `external-dns/`, etc.: Each directory is a self-contained component deployed as a Helm chart or Kubernetes manifests. Each has its own `README.md` for specific details.
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
    # Follow the instructions in the argocd/README.md for the initial ArgoCD installation.
    # Typically, this involves creating the namespace and applying the official Helm chart.
    ```

4.  **Bootstrap the App-of-Apps:**
    Once ArgoCD is running, you apply the root application, which tells ArgoCD to manage all other applications defined in this repository.
    ```sh
    # Ensure you are in the root of the repository
    kubectl apply -f argocd/argodc-applications.yaml
    ```
    ArgoCD will now start deploying all the components listed in `argodc-applications.yaml`. You can monitor the progress from the ArgoCD UI.

## Accessing Your Lab

*   **ArgoCD:** Follow the instructions in the `argocd/README.md` to get the initial admin password and access the UI, typically via a port-forward.
*   **Grafana, Headlamp, etc.:** Once deployed by ArgoCD, these applications will be accessible via Ingress. Check the hostnames defined in their respective `ingress.yaml` or `values.yaml` files.

## Components

| Component | Installation Method | Description |
| --- | --- | --- |
| [**ArgoCD**](argocd/) | Helm | Installs and configures ArgoCD for GitOps-style continuous deployment. |
| [**cert-manager**](cert-manager/) | Helm | Sets up cert-manager for automated TLS certificate management from various issuing sources. |
| [**ExternalDNS**](external-dns/) | Helm | Configures ExternalDNS to automatically manage DNS records for your services and ingresses. |
| [**IDP (authentik)**](idp/) | Helm | Deploys authentik as an Identity Provider for centralized authentication. |
| [**k8s-apiserver-oidc**](k8s-apiserver-oidc/) | Direct Apply | Configures the Kubernetes API server to use an OIDC provider for user authentication. |
| [**MetalLB**](metallb/) | Helm | Installs MetalLB to provide LoadBalancer services for your bare-metal cluster. |
| [**Monitoring**](monitoring/) | Helm | Sets up the kube-prometheus-stack for a complete cluster monitoring solution with Prometheus and Grafana. |
| [**Headlamp**](my-headlamp/) | Helm | Installs Headlamp, a web-based UI for Kubernetes. |
| [**StorageClass**](storageclass/) | Direct Apply | Defines a StorageClass for persistent storage using the NFS CSI driver. |
| [**Main Application**](base/) | Kustomize | The main application, deployed via ArgoCD. |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.