# ExternalDNS

This document provides instructions for installing and configuring ExternalDNS, a Kubernetes tool that synchronizes exposed Services and Ingresses with DNS providers.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- A Cloudflare API Token.

## Configuration

1.  **Seal your Cloudflare API token secret:**
    Create a `SealedSecret` for your Cloudflare API token. Do not commit the unencrypted `secrets.yaml` file.
    ```sh
    kubectl create secret generic cloudflare-api-token-secret \
      --namespace external-dns \
      --from-literal=api-token='YOUR_CLOUDFLARE_API_TOKEN' \
      --dry-run=client -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > infrastructure/external-dns/secrets-sealed.yaml
    ```
    After sealing, you can apply the sealed secret:
    ```sh
    kubectl apply -f infrastructure/external-dns/secrets-sealed.yaml
    ```

2.  **Configure the `values.yaml` file:**
    The `values.yaml` file configures ExternalDNS to use Cloudflare as the DNS provider and references the secret created in the previous step.

## Installation

1.  **Add the ExternalDNS Helm repository:**
    ```sh
    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
    helm repo update
    ```

2.  **Install ExternalDNS:**
    ```sh
    helm upgrade --install external-dns external-dns/external-dns --values infrastructure/external-dns/values.yaml
    ```
    This command installs ExternalDNS using the configuration from your `values.yaml` file.

## References

- [ExternalDNS GitHub Repository](https://github.com/kubernetes-sigs/external-dns)
