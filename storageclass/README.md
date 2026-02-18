# CSI NFS Installation Guide (Helm Method)

This guide provides the steps to install the CSI NFS (Container Storage Interface Network File System) driver on your Kubernetes cluster using Helm and an existing NFS server.

## Prerequisites

- A running Kubernetes cluster (v1.17 or later recommended)
- An accessible NFS server with a shared export
- `kubectl` configured to access your cluster
- [Helm](https://helm.sh/) installed

## Installation Steps

1. **Add the CSI NFS Helm Repository:**

    ```sh
    helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
    helm repo update
    ```

2. **Install the CSI NFS Driver:**

    Install a specific version (e.g., `4.12.0`) of the CSI NFS driver:

    ```sh
    helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version 4.12.0
    ```

    After installation, check the status of the CSI NFS Driver pods:

    ```sh
    kubectl --namespace=kube-system get pods --selector="app.kubernetes.io/instance=csi-driver-nfs"
    ```

    All controller and node pods should reach the `Running` state.

3. **Create a StorageClass:**

    Create a `StorageClass` that uses the NFS CSI driver. Replace `<NFS_SERVER>` and `<NFS_PATH>` with your actual NFS server address and export path.

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: nfs-csi
    provisioner: nfs.csi.k8s.io
    parameters:
      server: <NFS_SERVER>
      share: <NFS_PATH>
    ```

    Apply the StorageClass:

    ```sh
    kubectl apply -f storageclass.yaml
    ```

4. **Create a PersistentVolumeClaim (PVC):**

    Example PVC using the above StorageClass:

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs-pvc
    spec:
      accessModes:
        - ReadWriteMany
      storageClassName: nfs-csi
      resources:
        requests:
          storage: 10Gi
    ```

    Apply the PVC:

    ```sh
    kubectl apply -f pvc.yaml
    ```

5. **Use the PVC in your Pods:**

    Reference the `nfs-pvc` in your Pod or Deployment to mount the NFS volume.

## References

- [CSI NFS Driver Documentation](https://github.com/kubernetes-csi/csi-driver-nfs)
- [CSI NFS Helm Chart](https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts)