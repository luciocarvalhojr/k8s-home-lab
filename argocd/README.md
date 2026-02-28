# ArgoCD

This document outlines the installation and configuration of ArgoCD, a declarative GitOps continuous delivery tool for Kubernetes.

## 1. Installation

1.  **Add the ArgoCD Helm repository:**
    ```sh
    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update
    ```

2.  **Create the argocd namespace:**
    ```sh
    kubectl create namespace argocd
    ```

3.  **Install ArgoCD using Helm:**
    ```sh
    helm install argocd argo/argo-cd --namespace argocd -f argocd/values.yaml
    ```

## 2. Configuration

### OIDC Integration

If you are using an OIDC provider (like authentik), you should seal the client secret.

1.  **Seal your OIDC client secret:**
    Create a `SealedSecret` for your OIDC client secret.
    ```sh
    kubectl create secret generic argocd-oidc-secret \
      --namespace argocd \
      --from-literal=clientSecret='YOUR_OIDC_CLIENT_SECRET' \
      --dry-run=client -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > argocd/secrets-sealed.yaml
    ```
    After sealing, you can apply the sealed secret:
    ```sh
    kubectl apply -f argocd/secrets-sealed.yaml
    ```

2.  **Update `values.yaml`:**
    Ensure your `argocd/values.yaml` references this secret instead of containing the plain-text secret.

## 3. Expose the ArgoCD API Server

You can use either port-forwarding for a quick test or an IngressRoute for a permanent solution.

### Option 1: Port-forwarding

    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    Access ArgoCD at `https://localhost:8080`.