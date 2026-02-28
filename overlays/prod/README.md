# Production Overlay

This directory contains the Kustomize overlay for the production environment.

## Secrets

### Database Credentials

The production environment uses a sealed secret for database credentials.

To update or create the sealed secret:

```sh
kubectl create secret generic db-credentials 
  --namespace prod 
  --from-literal=username='admin' 
  --from-literal=password='YOUR_SAFE_PASSWORD' 
  --dry-run=client -o yaml | 
kubeseal 
  --controller-name=sealed-secrets 
  --controller-namespace=sealed-secrets 
  --format yaml > overlays/prod/secrets/secrets-db-sealed.yaml
```

Apply the sealed secret:

```sh
kubectl apply -f overlays/prod/secrets/secrets-db-sealed.yaml
```

## Configuration

The production overlay includes several patches:

- `host-patch.yaml`: Updates the ingress hostnames for production.
- `replica-patch.yaml`: Adjusts the number of replicas for production services.
