# cert-manager

This guide provides instructions for installing and configuring cert-manager, a tool for automating the management and issuance of TLS certificates in Kubernetes.

## Prerequisites

- A running Kubernetes cluster (v1.16+).
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Installation

1.  **Add the Jetstack Helm repository:**
    ```sh
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    ```

2.  **Install cert-manager:**
    ```sh
    helm install cert-manager jetstack/cert-manager \
      --namespace cert-manager \
      --create-namespace \
      --version v1.14.4 \
      --set installCRDs=true
    ```
    This command installs the cert-manager Helm chart, creating the `cert-manager` namespace and installing the required Custom Resource Definitions (CRDs).

3.  **Verify the installation:**
    ```sh
    kubectl get pods --namespace cert-manager
    ```
    You should see the `cert-manager`, `cert-manager-cainjector`, and `cert-manager-webhook` pods in a `Running` state.

## GoDaddy Webhook Installation

To use GoDaddy for DNS-01 challenges, you need to install the GoDaddy webhook.

1.  **Add the GoDaddy webhook Helm repo:**
    ```sh
    helm repo add godaddy-webhook https://snowdrop.github.io/godaddy-webhook
    ```

2.  **Install the webhook:**
    ```sh
    export DOMAIN=acme.mydomain.com # Replace with your domain
    helm install acme-webhook godaddy-webhook/godaddy-webhook \
      -n cert-manager \
      --set groupName=$DOMAIN
    ```

## Configuration

This project includes examples for creating a `ClusterIssuer` and a wildcard `Certificate`.

1.  **Create a secret with your Cloudflare API token:**
    Modify the `secret.yaml` file with your Cloudflare API token and apply it.
    ```sh
    kubectl apply -f cert-manager/secret.yaml
    ```

2.  **Create the ClusterIssuer:**
    The `clusterissuer.yaml` file defines a `ClusterIssuer` that uses Cloudflare to solve DNS-01 challenges.
    ```sh
    kubectl apply -f cert-manager/clusterissuer.yaml
    ```

3.  **Create a wildcard certificate:**
    The `certificate-wildcard.yaml` file defines a wildcard certificate for your domain.
    ```sh
    kubectl apply -f cert-manager/certificate-wildcard.yaml
    ```

## Uninstallation

1.  **Uninstall the Helm chart:**
    ```sh
    helm uninstall cert-manager -n cert-manager
    ```

2.  **Delete the CRDs:**
    ```sh
    kubectl delete crd \
      issuers.cert-manager.io \
      clusterissuers.cert-manager.io \
      certificates.cert-manager.io \
      certificaterequests.cert-manager.io \
      orders.acme.cert-manager.io \
      challenges.acme.cert-manager.io
    ```

## References

- [cert-manager Documentation](https://cert-manager.io/docs/)
- [GoDaddy Webhook for cert-manager](https://github.com/snowdrop/godaddy-webhook)
