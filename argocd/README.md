# ArgoCD on k3s (Little Cluster)

This guide helps you install ArgoCD on your k3s cluster and configure access using Traefik.

## 1. Install ArgoCD
Apply the official manifest:

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 2. Expose ArgoCD API Server

### Option 1: Port-forwarding (quick test)
```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access at: https://localhost:8080

### Option 2: Traefik IngressRoute (recommended)
Apply the provided ingress configuration:

```sh
kubectl apply -f argocd/argocd-ingress.yaml
```

Make sure your DNS or `/etc/hosts` points `argocd.k3s.home.lan` to your k3s cluster IP.

## 3. Get the Initial Admin Password
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## 4. Access the ArgoCD UI
Go to your ingress address (e.g., https://argocd.k3s.home.lan) or https://localhost:8080 if using port-forward.

Login with username `admin` and the password from the previous step.

---
For more advanced configuration, see the [official docs](https://argo-cd.readthedocs.io/en/stable/getting_started/).
```

```