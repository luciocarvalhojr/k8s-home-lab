# my-headlamp

A simple guide to install Headlamp on your Kubernetes cluster.

## Prerequisites

- Kubernetes cluster access (`kubectl` configured)
- Helm installed

## Installation

1. **Add the Headlamp Helm repo:**
    ```sh
    helm repo add headlamp https://headlamp-k8s.github.io/headlamp/
    helm repo update
    ```

2. **Install Headlamp:**
    ```sh
    helm install my-headlamp headlamp/headlamp
    ```

3. **Access Headlamp:**
    ```sh
    kubectl port-forward svc/my-headlamp-headlamp 8080:80
    ```
    Open [http://localhost:8080](http://localhost:8080) in your browser.

4. **Get the token using:**
    ```sh
  kubectl create token my-headlamp --namespace kube-system
    ```
## Uninstall

```sh
helm uninstall my-headlamp
```

## References

- [Headlamp Documentation](https://headlamp.dev/)