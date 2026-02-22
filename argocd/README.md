# ArgoCD

This guide helps you install ArgoCD on your k3s cluster and configure access using Traefik.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Traefik installed as an Ingress Controller (for IngressRoute access).

## Installation

1.  **Create the ArgoCD namespace:**
    ```sh
    kubectl create namespace argocd
    ```

2.  **Apply the official manifest:**
    ```sh
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

3.  **Expose the ArgoCD API Server.**

    You can use either port-forwarding for a quick test or an IngressRoute for a permanent solution.

    *   **Option 1: Port-forwarding**
        ```sh
        kubectl port-forward svc/argocd-server -n argocd 8080:443
        ```
        Access ArgoCD at `https://localhost:8080`.

    *   **Option 2: Traefik IngressRoute**
        Apply the provided `argocd-ingressroute.yaml` configuration.
        ```sh
        kubectl apply -f argocd/argocd-ingressroute.yaml
        ```
        Ensure your DNS is configured to point `argocd.k3s.home.lan` to your cluster's IP.

## Accessing ArgoCD

1.  **Get the initial admin password:**
    ```sh
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
    ```

2.  **Access the ArgoCD UI.**
    Open your Ingress address (e.g., `https://argocd.k3s.home.lan`) or `https://localhost:8080` if using port-forwarding. Login with the username `admin` and the password retrieved in the previous step.

## References

- [ArgoCD Official Documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/)
