# k8s-home-lab

This repository contains a collection of tutorials and configurations for setting up a Kubernetes home lab. Each directory in this repository represents a component of the home lab and contains the necessary Kubernetes manifests and a README file with step-by-step instructions.

## Table of Contents

- [ArgoCD](#argocd)
- [Cert-Manager](#cert-manager)
- [ExternalDNS](#externaldns)
- [MetalLB](#metallb)
- [Monitoring](#monitoring)
- [Headlamp](#headlamp)
- [StorageClass](#storageclass)

## Components

### ArgoCD

Installs ArgoCD and configures it with Traefik for access.

- [ArgoCD README](argocd/README.md)

### Cert-Manager

Sets up Cert-Manager for automatic TLS certificate management.

- [Cert-Manager README](cert-manager/README.md)

### ExternalDNS

Configures ExternalDNS to manage DNS records for your services.

- [ExternalDNS README](external-dns/README.md)

### MetalLB

Installs MetalLB to provide LoadBalancer services for your bare-metal cluster.

- [MetalLB README](metallb/README.md)

### Monitoring

Sets up Prometheus and Grafana for monitoring your cluster.

- [Monitoring README](monitoring/README.md)

### Headlamp

Installs Headlamp, a web UI for Kubernetes.

- [Headlamp README](my-headlamp/README.md)

### StorageClass

Defines a StorageClass for persistent storage using NFS.

- [StorageClass README](storageclass/README.md)

## Usage

Follow the instructions in the README file of each component to set up your home lab. You can start with MetalLB and then proceed with the other components.