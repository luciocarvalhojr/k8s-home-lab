# NFS CSI Driver

This document provides instructions for installing and configuring the NFS CSI (Container Storage Interface) driver, allowing you to use an existing NFS server for persistent storage in your Kubernetes cluster.

## Prerequisites

- A running Kubernetes cluster (v1.17+).
- An accessible NFS server with a shared export.
- `kubectl` configured to access the cluster.
- Helm v3 or later.

## Installation and Configuration

1.  **Add the NFS CSI driver Helm repository:**
    ```sh
    helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
    helm repo update
    ```

2.  **Install the NFS CSI driver:**
    ```sh
    helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version 4.12.0
    ```
    Verify that the driver pods are running in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system -l "app.kubernetes.io/instance=csi-driver-nfs"
    ```

3.  **Create the StorageClass:**
    The `storageclass/storageclass.yaml` file defines a `StorageClass` that uses the NFS CSI driver. Modify the `server` and `share` parameters in the file to point to your NFS server. Then, apply the configuration.
    ```sh
    kubectl apply -f storageclass/storageclass.yaml
    ```

4.  **Create a PersistentVolumeClaim (PVC):**
    The `storageclass/pvc-example.yaml` file provides an example of how to create a `PersistentVolumeClaim` that uses the `nfs-csi` StorageClass.
    ```sh
    kubectl apply -f storageclass/pvc-example.yaml
    ```
    You can now reference this PVC in your Pods to mount the NFS volume.

## References

- [NFS CSI Driver GitHub Repository](https://github.com/kubernetes-csi/csi-driver-nfs)
