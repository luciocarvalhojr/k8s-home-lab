# Monitoring Stack

This document provides instructions for installing the kube-prometheus-stack, which includes Prometheus, Grafana, and other monitoring components.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Configuration

This project includes example files for `values.yaml` and `secrets-sealed.yaml`.

1.  **Seal your Grafana Authentik secret:**
    Create a `SealedSecret` for your Grafana Authentik integration. Do not commit the unencrypted `secrets.yaml` file.
    ```sh
    kubectl create secret generic grafana-authentik \
      --namespace monitoring \
      --from-literal=client_secret='YOUR_AUTHENTIK_CLIENT_SECRET' \
      --dry-run=client -o yaml | \
    kubeseal \
      --controller-name=sealed-secrets \
      --controller-namespace=sealed-secrets \
      --format yaml > infrastructure/monitoring/secrets-sealed.yaml
    ```
    After sealing, you can apply the sealed secret:
    ```sh
    kubectl apply -f infrastructure/monitoring/secrets-sealed.yaml
    ```

2.  **Configure `values.yaml`:**
    The `infrastructure/monitoring/values.yaml` file contains the configuration for the kube-prometheus-stack Helm chart. Review and customize it as needed.

## Installation

1.  **Add the Prometheus Community Helm repository:**
    ```sh
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    ```

2.  **Create the monitoring namespace:**
    ```sh
    kubectl create namespace monitoring
    ```

3.  **Install the kube-prometheus-stack:**
    ```sh
    helm install monitoring prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      -f infrastructure/monitoring/values.yaml
    ```

4.  **Verify the installation:**
    ```sh
    kubectl get pods -n monitoring -l "release=monitoring"
    ```

## Accessing Services

### Grafana

1.  **Get the Grafana admin password:**
    ```sh
    kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    ```

2.  **Port-forward the Grafana service:**
    ```sh
    kubectl port-forward --namespace monitoring svc/monitoring-grafana 3000:80
    ```
    Access Grafana at `http://localhost:3000`. Login with the username `admin` and the password from the previous step.

### Prometheus

1.  **Port-forward the Prometheus service:**
    ```sh
    kubectl port-forward --namespace monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
    ```
    Access Prometheus at `http://localhost:9090`.

## Uninstallation

```sh
helm uninstall monitoring --namespace monitoring
kubectl delete namespace monitoring
```

## References

- [kube-prometheus-stack Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
