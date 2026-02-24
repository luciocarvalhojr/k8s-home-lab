# k8s-home-lab

This repository contains a collection of configurations for setting up a Kubernetes home lab. Each directory represents a component and contains the necessary Kubernetes manifests and a README with step-by-step instructions.

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

## Getting Started

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/your-username/k8s-home-lab.git
    cd k8s-home-lab
    ```

2.  **Review the components:**
    Each component is designed to be self-contained. Read the `README.md` in each directory to understand its purpose and how to deploy it.

3.  **Deploy the components:**
    Follow the instructions in each component's `README.md` to deploy it to your cluster. It is recommended to start with `MetalLB` and `StorageClass` to provide the foundational networking and storage layers.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
