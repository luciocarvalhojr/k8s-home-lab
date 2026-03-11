# Headlamp

This document provides instructions for installing Headlamp, a web-based UI for Kubernetes.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Configuration

This project includes example files for `values.yaml` and `ingress.yaml`.

1.  **Seal your OIDC client secret:**

    > **Warning:** `apps/my-headlamp/values.yaml` currently contains `config.oidc.clientSecret` in plaintext. This must be sealed before committing to a public repository.

    Seal the value using the cluster's public key:
    ```sh
    echo -n "YOUR_OIDC_CLIENT_SECRET" | kubeseal --raw \
      --name headlamp-oidc-secret \
      --namespace default \
      --cert sealed-secrets.pem
    ```

    Create `apps/my-headlamp/secrets-sealed.yaml` with the encrypted value as a `SealedSecret` resource, following the same pattern as `apps/observatory/auth-svc-sealed-secret.yaml`.

2.  **Configure `values.yaml`:**
    The `apps/my-headlamp/values.yaml` file contains the configuration for the Headlamp Helm chart. Review and customize it as needed.

3.  **Configure Ingress:**
    The `apps/my-headlamp/ingress.yaml` file provides an example of how to expose Headlamp using an Ingress.

## Installation and Access

1.  **Add the Headlamp Helm repository:**
    ```sh
    helm repo add headlamp https://headlamp-k8s.github.io/headlamp/
    helm repo update
    ```

2.  **Install Headlamp:**
    ```sh
    helm install my-headlamp headlamp/headlamp -f apps/my-headlamp/values.yaml
    ```

3.  **Access Headlamp:**
    You can access Headlamp either by port-forwarding the service or by using the Ingress.

    *   **Option 1: Port-forwarding**
        ```sh
        kubectl port-forward svc/my-headlamp 8080:80
        ```
        Access Headlamp at `http://localhost:8080`.

    *   **Option 2: Ingress**
        Apply the `ingress.yaml` file.
        ```sh
        kubectl apply -f apps/my-headlamp/ingress.yaml
        ```
        Access Headlamp at the hostname specified in the Ingress.

4.  **Get an authentication token:**
    ```sh
    kubectl create token my-headlamp-sa --namespace default # Or the correct service account
    ```
    Use this token to log in to the Headlamp UI.

## Uninstallation

```sh
helm uninstall my-headlamp
```

## References

- [Headlamp Documentation](https://headlamp.dev/)
