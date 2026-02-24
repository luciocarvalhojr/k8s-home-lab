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

## 2. Expose the ArgoCD API Server

You can use either port-forwarding for a quick test or an IngressRoute for a permanent solution.

### Option 1: Port-forwarding

    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    Access ArgoCD at `https://localhost:8080`.