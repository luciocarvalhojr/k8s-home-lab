# Headlamp

This document provides instructions for installing Headlamp, a web-based UI for Kubernetes.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Configuration

This project includes example files for `values.yaml` and `ingress.yaml`.

1.  **Configure `values.yaml`:**
    The `my-headlamp/values.yaml` file contains the configuration for the Headlamp Helm chart. Review and customize it as needed.

2.  **Configure Ingress:**
    The `my-headlamp/ingress.yaml` file provides an example of how to expose Headlamp using an Ingress.

## Installation and Access

1.  **Add the Headlamp Helm repository:**
    ```sh
    helm repo add headlamp https://headlamp-k8s.github.io/headlamp/
    helm repo update
    ```

2.  **Install Headlamp:**
    ```sh
    helm install my-headlamp headlamp/headlamp -f my-headlamp/values.yaml
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
        kubectl apply -f my-headlamp/ingress.yaml
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
