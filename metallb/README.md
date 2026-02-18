# MetalLB Installation Guide for K3s

This guide provides steps to install MetalLB on a K3s cluster using Helm, including instructions to disable the default ServiceLB and configure MetalLB via a `values.yaml` file.

## 1. Disable the Default ServiceLB in K3s

To prevent conflicts with MetalLB, disable the built-in ServiceLB by editing the K3s systemd service:

1. Edit the K3s service file (usually `/etc/systemd/system/k3s.service` or `/etc/systemd/system/k3s.service.env`).
2. Add the following flag to the K3s server arguments:
    ```
    --disable servicelb
    ```
3. Reload the systemd daemon and restart K3s:
    ```
    sudo systemctl daemon-reload
    sudo systemctl restart k3s
    ```

## 2. Add the MetalLB Helm Repository
Add the MetalLB Helm repository and update your Helm repos:

```sh
helm repo add metallb https://metallb.github.io/metallb
helm repo update
```

## 3. Create a Namespace for MetalLB

It's recommended to install MetalLB in its own namespace:

```sh
kubectl create namespace metallb-system
```

## 4. Configure MetalLB Address Pool

Create a `values.yaml` file to define the address pool MetalLB will use. Example:

```yaml
configInline:
    address-pools:
    - name: default
        protocol: layer2
        addresses:
        - 192.168.1.240-192.168.1.250
```

Adjust the IP range to match your network.

## 5. Install MetalLB with Helm

Install MetalLB using your custom `values.yaml`:

```sh
helm install metallb metallb/metallb \
    --namespace metallb-system \
    --values values.yaml
```

## 6. Verify MetalLB Installation

Check that the MetalLB pods are running:

```sh
kubectl get pods -n metallb-system
```

## 7. Configure Traefik to Use MetalLB

If Traefik is installed as the default ingress controller in K3s, update its Service to use `LoadBalancer` type:

```sh
kubectl -n kube-system edit svc traefik
```

Change the `type` field to `LoadBalancer`:

```yaml
spec:
    type: LoadBalancer
```

Save and exit the editor. Traefik will now receive an external IP from MetalLB.

## 8. Test the Setup

After a few moments, check that Traefik has an external IP:

```sh
kubectl get svc -n kube-system traefik
```

You should see an external IP from the range you configured in MetalLB.

---
You have now installed MetalLB on your K3s cluster and configured Traefik to use it for external access.