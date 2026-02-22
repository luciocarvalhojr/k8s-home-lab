# authentik (IDP)

This document provides instructions for installing authentik, an open-source Identity Provider (IDP), using the official Helm chart.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- An Ingress controller (like Traefik or NGINX) installed.

## Configuration

The `idp/values.yaml` file contains the configuration for the authentik Helm chart. You should review and customize it to fit your environment. At a minimum, you must set the following values:

-   `authentik.secret_key`: A securely generated secret key.
-   `authentik.postgresql.password`: A strong password for the PostgreSQL database.
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