# Prometheus & Grafana Installation Guide

This guide explains how to install Prometheus and Grafana in your Kubernetes cluster using the kube-prometheus-stack Helm chart.

## Prerequisites

- Kubernetes cluster up and running
- `kubectl` configured to access your cluster
- [Helm](https://helm.sh/) installed

## Installation Steps

### 1. Add Prometheus Community Helm Repo

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 2. Create Monitoring Namespace

```sh
kubectl create namespace monitoring
```

### 3. Install kube-prometheus-stack

```sh
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring
```

This will install Prometheus, Grafana, and related monitoring components with default settings.

### 4. Verify Installation

```sh
kubectl --namespace monitoring get pods -l "release=monitoring"
```

### 5. Access Grafana UI

Get the Grafana admin password:

```sh
kubectl --namespace monitoring get secrets monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

Port-forward the Grafana pod:

```sh
export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=monitoring" -oname)
kubectl --namespace monitoring port-forward $POD_NAME 3000
```

Then open [http://localhost:3000](http://localhost:3000) in your browser.  
Login with username `admin` and the password you retrieved above.

### 6. Access Prometheus UI

Port-forward the Prometheus server:

```sh
kubectl --namespace monitoring port-forward svc/monitoring-kube-prometheus-stack-prometheus 9090
```

Then open [http://localhost:9090](http://localhost:9090) in your browser.

## Customization

To customize your installation, create a `values.yaml` file and install with:

```sh
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring -f ingress.yaml
```

## Uninstall

```sh
helm uninstall monitoring --namespace monitoring
kubectl delete namespace monitoring
```

---

For more details, see the [kube-prometheus-stack Helm Chart documentation](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).