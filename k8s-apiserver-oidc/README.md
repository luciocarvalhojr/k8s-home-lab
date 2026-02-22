# k8s-apiserver-oidc

This document outlines the steps to configure OIDC authentication for the Kubernetes API server, using an external Identity Provider (like authentik).

## Prerequisites

- A running Kubernetes cluster (e.g., k3s).
- An OIDC provider (e.g., authentik) configured with a client ID and secret.
- `kubectl` with `oidc-login` plugin installed (`kubectl krew install oidc-login`).

## 1. authentik Configuration

### A. Create the Provider

1.  Navigate to **Applications -> Providers -> Create**.
2.  Select **OAuth2/OpenID Provider**.
3.  **Name**: `kube-apiserver`
4.  **Authorization flow**: Default (e.g., explicit-consent).
5.  **Client Type**: Confidential.
6.  **Redirect URIs**: `http://localhost:8000` and `http://localhost:18000` (for `kubectl oidc-login`).
7.  **Issuer Mode**: Per-application.

### B. Fix the "Email Verified" Claim

Kubernetes requires `email_verified: true`. If your users aren't manually verified, create a custom mapping:

1.  Go to **Customization -> Property Mappings**.
2.  Create a **Scope Mapping**:
    *   **Name**: `OIDC-K8s-Email-Fix`
    *   **Scope name**: `email`
    *   **Expression**:
        ```python
        return {
            "email": request.user.email,
            "email_verified": True
        }
        ```
3.  Attach this mapping to your Provider under **Advanced Protocol Settings**.

### C. Enable Groups Mapping

Ensure the `authentik default OAuth Mapping: OpenID 'groups'` is selected in your Provider's **Property Mappings**.

## 2. k3s Server Configuration

Update your k3s configuration (`/etc/rancher/k3s/config.yaml`) to enable OIDC.

```yaml
kube-apiserver-arg:
  - "oidc-issuer-url=https://auth.capihome.xyz/application/o/kube-apiserver/"
  - "oidc-client-id=kube-apiserver"
  - "oidc-username-claim=email"
  - "oidc-groups-claim=groups"
```

After editing, restart the k3s service:
```sh
sudo systemctl restart k3s
```

## 3. RBAC Configuration

Apply the `ClusterRoleBinding` to grant `cluster-admin` rights to an OIDC group.

```sh
kubectl apply -f k8s-apiserver-oidc/clusterrolebinding-oidc-group-kube-apiserver-admins.yaml
```
This binds the `cluster-admin` role to the `kube-apiserver-admins` group, which must exist in your OIDC provider.

## 4. Client Setup (Local Machine)

Configure `kubectl` to use the OIDC provider for a user.

```bash
kubectl config set-credentials capivara \
  --exec-api-version=client.authentication.k8s.io/v1beta1 \
  --exec-command=kubectl \
  --exec-arg=oidc-login \
  --exec-arg=get-token \
  --exec-arg=--oidc-issuer-url=https://auth.capihome.xyz/application/o/kube-apiserver/ \
  --exec-arg=--oidc-client-id=kube-apiserver \
  --exec-arg=--oidc-client-secret=YOUR_CLIENT_SECRET \
  --exec-arg=--oidc-extra-scope="email profile groups"
```

To use it, run a command with the specified user:
```sh
kubectl get nodes --user=capivara
```

## Troubleshooting

- **Unauthorized:** Ensure your OIDC provider returns `email_verified: true`.
- **Forbidden:** Double-check that the group name in `clusterrolebinding-oidc-group-kube-apiserver-admins.yaml` matches the group claim from your OIDC provider exactly.
- **Invalid Token:** Sync the time between your k3s nodes and your OIDC provider.