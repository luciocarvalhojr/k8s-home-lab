# MetalLB

This document provides instructions for installing and configuring MetalLB, a load-balancer for bare metal Kubernetes clusters.

## Prerequisites

- A running K3s cluster.
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Installation and Configuration

1.  **Disable the K3s ServiceLB:**
    To avoid conflicts, disable the default K3s load balancer. Edit `/etc/systemd/system/k3s.service` and add the `--disable servicelb` flag to the server arguments. Then, restart the K3s service.
    ```sh
    sudo systemctl daemon-reload
    sudo systemctl restart k3s
    ```

2.  **Add the MetalLB Helm repository:**
    ```sh
    helm repo add metallb https://metallb.github.io/metallb
    helm repo update
    ```

3.  **Create the MetalLB namespace:**
    ```sh
    kubectl create namespace metallb-system
    ```

4.  **Install MetalLB:**
    ```sh
    helm install metallb metallb/metallb --namespace metallb-system
    ```

5.  **Configure the address pool:**
    Modify the `metallb/configmap.yaml` file to define the IP address range that MetalLB will manage. Then, apply the configuration.
    ```sh
    kubectl apply -f metallb/configmap.yaml -n metallb-system
    ```

6.  **Verify the installation:**
    Check that the MetalLB pods are running in the `metallb-system` namespace.
    ```sh
    kubectl get pods -n metallb-system
    ```

## Using MetalLB with Traefik

To expose the Traefik Ingress controller, change its Service type to `LoadBalancer`.
```sh
kubectl -n kube-system patch svc traefik -p '{"spec":{"type":"LoadBalancer"}}'
```
MetalLB will assign an external IP from its pool to the Traefik Service.

## References

- [MetalLB Website](https://metallb.universe.tf/)
