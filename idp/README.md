# authentik (IDP)

This document provides instructions for installing authentik, an open-source Identity Provider (IDP), using the official Helm chart.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- An Ingress controller (like Traefik or NGINX) installed.

## Configuration

The `idp/values.yaml` file contains the configuration for the authentik Helm chart. You should review and customize it to fit your environment.

1.  **Seal your authentik secrets:**
    At a minimum, you must set the `authentik.secret_key` and `authentik.postgresql.password`. Instead of putting them in `values.yaml`, it is recommended to seal them.
    ```sh
    kubectl create secret generic authentik-secrets \
      --namespace default \
      --from-literal=secret_key='YOUR_AUTHENTIK_SECRET_KEY' \
      --from-literal=postgresql_password='YOUR_POSTGRES_PASSWORD' \
      --dry-run=client -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > idp/secrets-sealed.yaml
    ```
    After sealing, you can apply the sealed secret:
    ```sh
    kubectl apply -f idp/secrets-sealed.yaml
    ```

2.  **Configure `values.yaml`:**
    Modify the `idp/values.yaml` file to reference these secrets if supported by the chart, or ensure you are not committing plain-text secrets there. At a minimum, set:
    -   `server.ingress.hosts`: The hostname for accessing the authentik UI.

## Installation

1.  **Add the authentik Helm repository:**
    ```sh
    helm repo add authentik https://charts.goauthentik.io
    helm repo update
    ```

2.  **Install authentik:**
    ```sh
    helm upgrade --install authentik authentik/authentik -f idp/values.yaml
    ```
    This command installs authentik using the configuration from your `values.yaml` file.

## Accessing authentik

After the installation is complete, access the initial setup at `https://<your-ingress-host>/if/flow/initial-setup/` to create the `akadmin` user and set a password.

> **Note:** Ensure you include the trailing slash in the URL.

## References

- [authentik Website](https://goauthentik.io/)
- [authentik Helm Chart on ArtifactHub](https://artifacthub.io/packages/helm/authentik/authentik)