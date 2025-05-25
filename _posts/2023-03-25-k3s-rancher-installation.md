---
title: "K3S and Rancher: A Complete Installation Guide"
date: 2023-03-25 19:10:00 +0700
categories: ["DevOps", "Kubernetes"]
tags: ["DevOps", "Kubernetes", "K3S", "Rancher", "Installation"]
pin: false
---

# K3S and Rancher: A Complete Installation Guide

K3S and Rancher together provide a lightweight yet powerful solution for managing Kubernetes clusters. This guide walks you through the installation process, from prerequisites to deploying Rancher with SSL.

## Why Choose K3S and Rancher?

### K3S

K3S is a lightweight, certified Kubernetes distribution designed for resource-constrained environments like edge computing, IoT, and ARM-based systems. It simplifies Kubernetes deployment while maintaining compatibility with standard Kubernetes tools.

### Rancher

Rancher is a comprehensive platform for managing multiple Kubernetes clusters. It offers centralized management, monitoring, and automation tools, making it easier to deploy and scale containerized applications.

Together, K3S and Rancher provide a robust solution for Kubernetes management, especially in environments with limited resources.

## Prerequisites

Before you begin, ensure you have:

- A server or Raspberry Pi with a supported operating system.
- Root or sudo access to the server.
- A domain name for accessing Rancher.

## Step-by-Step Installation

### 1. Configure Raspberry Pi (if applicable)

For Raspberry Pi, enable memory cgroups by editing `/boot/firmware/cmdline.txt`:

```ini
console=serial0,115200 console=tty1 root=PARTUUID=96955a5f-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=VN cgroup_memory=1 cgroup_enable=memory
```

### 2. Install K3S, Helm, and cert-manager

Use the following script to install K3S, Helm, and cert-manager:

```bash
#!/bin/bash

# Set K3S_VERSION to the desired version
export INSTALL_K3S_VERSION=v1.27.11+k3s1
export DOMAIN="example.com"
export K3S_EMAIL="admin@${DOMAIN}"
export RANCHER_HOSTNAME="rancher.${DOMAIN}"
export SSL_CERTIFICATE="/etc/letsencrypt/live/${DOMAIN}/fullchain.pem"
export SSL_PRIVATE_KEY="/etc/letsencrypt/live/${DOMAIN}/privkey.pem"
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Install K3S
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=${INSTALL_K3S_VERSION} sh -

# Wait for K3S to start up
sleep 5

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install cert-manager
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

### 3. Add Rancher Repository

Add the Rancher Helm repository:

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
```

### 4. Install Rancher

#### Option 1: Using Let's Encrypt

```bash
helm install rancher rancher-stable/rancher \
  --create-namespace \
  --namespace cattle-system \
  --set hostname=${RANCHER_HOSTNAME} \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=${K3S_EMAIL} \
  --set letsEncrypt.ingress.class=traefik \
  --set replicas=1
```

#### Option 2: Using External SSL Certificates

```bash
helm install rancher rancher-stable/rancher \
  --create-namespace \
  --namespace cattle-system \
  --set hostname=${RANCHER_HOSTNAME} \
  --set ingress.tls.source=secret

kubectl -n cattle-system create secret tls tls-rancher-ingress \
  --cert="${SSL_CERTIFICATE}" \
  --key="${SSL_PRIVATE_KEY}"
```

## Conclusion

By following this guide, you can set up K3S and Rancher to manage Kubernetes clusters efficiently. This setup is ideal for resource-constrained environments and provides a scalable solution for containerized workloads.
