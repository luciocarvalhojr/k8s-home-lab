# authentik (IDP)

This document provides instructions for installing authentik, an open-source Identity Provider (IDP), using the official Helm chart.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- An Ingress controller (like Traefik or NGINX) installed.

## Configuration

The `identity/idp/values.yaml` file contains the configuration for the authentik Helm chart. You should review and customize it to fit your environment.

1.  **Seal your authentik secrets:**

    > **Warning:** `identity/idp/values.yaml` currently contains `authentik.secret_key` and `authentik.postgresql.password` in plaintext. These must be sealed before committing to a public repository.

    Seal each value using the cluster's public key:
    ```sh
    echo -n "YOUR_AUTHENTIK_SECRET_KEY" | kubeseal --raw \
      --name authentik-secrets \
      --namespace idp \
      --cert sealed-secrets.pem

    echo -n "YOUR_POSTGRES_PASSWORD" | kubeseal --raw \
      --name authentik-secrets \
      --namespace idp \
      --cert sealed-secrets.pem
    ```

    Create `identity/idp/secrets-sealed.yaml` with the encrypted values as a `SealedSecret` resource, following the same pattern as `infrastructure/cert-manager/secrets-sealed.yaml` or `apps/observatory/auth-svc-sealed-secret.yaml`.

2.  **Configure `values.yaml`:**
    Modify the `identity/idp/values.yaml` file to reference secrets instead of containing plain-text values. At a minimum, set:
    -   `server.ingress.hosts`: The hostname for accessing the authentik UI (currently `auth.capihome.xyz`).

## Installation

1.  **Add the authentik Helm repository:**
    ```sh
    helm repo add authentik https://charts.goauthentik.io
    helm repo update
    ```

2.  **Install authentik:**
    ```sh
    helm upgrade --install authentik authentik/authentik -f identity/idp/values.yaml
    ```
    This command installs authentik using the configuration from your `values.yaml` file.

## Accessing authentik

After the installation is complete, access the initial setup at `https://<your-ingress-host>/if/flow/initial-setup/` to create the `akadmin` user and set a password.

> **Note:** Ensure you include the trailing slash in the URL.

## References

- [authentik Website](https://goauthentik.io/)
- [authentik Helm Chart on ArtifactHub](https://artifacthub.io/packages/helm/authentik/authentik)