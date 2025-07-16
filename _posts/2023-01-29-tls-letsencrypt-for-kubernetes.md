---
title: "Setting Up SSL on Kubernetes with cert-manager and Let's Encrypt"
date: 2023-01-29 19:10:00 +0700
categories: ["DevOps", "Containers"]
tags: ["DevOps", "Kubernetes", "SSL", "Let's Encrypt", "cert-manager"]
pin: false
---

# Setting Up SSL on Kubernetes with cert-manager and Let's Encrypt

Securing your Kubernetes applications with HTTPS is essential for protecting data and ensuring trust. This guide walks you through setting up SSL certificates on Kubernetes using `cert-manager` and Let's Encrypt.

## Why Use SSL on Kubernetes?

SSL (Secure Sockets Layer) encrypts data transmitted between clients and servers, ensuring privacy and security. On Kubernetes, SSL is crucial for:

- Securing ingress traffic.
- Protecting sensitive data.
- Complying with security standards.

## Options for SSL Certificates

There are two main options for obtaining SSL certificates:

### 1. Purchase Certificates

- **Pros**:
  - Simple installation.
  - Long validity (up to 5 years).
- **Cons**:
  - Costly.
  - Requires manual renewal.

### 2. Use Let's Encrypt

Let's Encrypt provides free SSL certificates with a validity of 3 months. Certificates can be automatically renewed using tools like `cert-manager`.

- **Pros**:
  - Free.
  - Automated renewal.
- **Cons**:
  - Complex initial setup.
  - Short validity period.

## Prerequisites

Before you begin, ensure you have:

- A Kubernetes cluster with ingress enabled.
- `kubectl` installed and configured.
- A domain name pointing to your cluster.

## Step-by-Step Guide

### 1. Install cert-manager

Install `cert-manager` using Helm:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

### 2. Configure Let's Encrypt Issuer

Create a ClusterIssuer for Let's Encrypt:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply the configuration:

```bash
kubectl apply -f cluster-issuer.yaml
```

### 3. Create a Certificate

Define a Certificate resource:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com-tls
  namespace: default
spec:
  secretName: example-com-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: example.com
  dnsNames:
  - example.com
  - www.example.com
```

Apply the configuration:

```bash
kubectl apply -f certificate.yaml
```

### 4. Verify the Setup

Check the status of the Certificate:

```bash
kubectl describe certificate example-com-tls
```

Ensure the secret is created:

```bash
kubectl get secrets
```

## Conclusion

By following this guide, you can secure your Kubernetes applications with SSL using `cert-manager` and Let's Encrypt. This setup ensures automated certificate management, providing a reliable and cost-effective solution for HTTPS on Kubernetes.
