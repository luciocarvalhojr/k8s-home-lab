# Sealed Secrets

This document provides instructions for installing and configuring Sealed Secrets, a tool from Bitnami Labs that allows you to store sensitive information in your Git repository by encrypting it into a `SealedSecret` resource.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.
- `kubeseal` CLI tool installed on your local machine.

## Installation

1.  **Add the Sealed Secrets Helm repository:**
    ```sh
    helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
    helm repo update
    ```

2.  **Install Sealed Secrets:**
    ```sh
    helm install sealed-secrets sealed-secrets/sealed-secrets \
        --namespace sealed-secrets --create-namespace \
        -f sealedsecrets/values.yaml
    ```
    This command installs the Sealed Secrets controller in the `sealed-secrets` namespace.

## Usage

### 1. Fetching the Public Certificate (Optional)

If you want to seal secrets without direct access to the cluster, you can fetch the public certificate used for encryption:

```sh
kubeseal \
  --controller-name=sealed-secrets \
  --controller-namespace=sealed-secrets \
  --fetch-cert > sealed-secrets.pem
```

### 2. Creating a Sealed Secret

To create a sealed secret, pipe a standard Kubernetes secret into `kubeseal`:

```sh
kubectl create secret generic secret-name \
  --dry-run=client \
  --from-literal=foo=bar \
  -o yaml | \
kubeseal \
  --controller-name=sealed-secrets \
  --controller-namespace=sealed-secrets \
  --format yaml > mysealedsecret.yaml
```

The resulting `mysealedsecret.yaml` can be safely committed to your version control system.

### 3. Cluster-wide Sealed Secret Example

To create a sealed secret that can be decrypted in any namespace (if allowed by the controller):

```sh
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='S3cr3t!' \
  --dry-run=client -o yaml | \
kubeseal --cert sealed-secrets.pem --scope cluster-wide \
        --format yaml > base/secrets-db-sealed.yaml
```

### 4. Apply the Sealed Secret

Apply the sealed secret to your cluster like any other Kubernetes resource:

```sh
kubectl apply -f mysealedsecret.yaml
```

## References

- [Sealed Secrets GitHub Repository](https://github.com/bitnami-labs/sealed-secrets)
- [Installation Guide](https://github.com/bitnami-labs/sealed-secrets#installation)
