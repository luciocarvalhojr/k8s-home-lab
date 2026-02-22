# ExternalDNS

This document provides instructions for installing and configuring ExternalDNS, a Kubernetes tool that synchronizes exposed Services and Ingresses with DNS providers.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- A Cloudflare API Token.

## Configuration

1.  **Create a secret with your Cloudflare API Token:**
    Modify the `secrets.yaml` file with your Cloudflare API token and apply it.
    ```sh
    kubectl apply -f external-dns/secrets.yaml
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
    helm upgrade --install external-dns external-dns/external-dns --values external-dns/values.yaml
    ```
    This command installs ExternalDNS using the configuration from your `values.yaml` file.

## References

- [ExternalDNS GitHub Repository](https://github.com/kubernetes-sigs/external-dns)
