---
title: K3S & Rancher Installation Script
date: 2023-03-25 19:10:00 +0700
categories: [Devops, Kubernetes]
tags: [Devops, Kubernetes, K3S, Rancher]
pin: false
---

## Why K3S and Rancher?

K3s is a lightweight, certified Kubernetes distribution designed for production workloads in resource-constrained environments such as edge, IoT, and ARM-based systems. It was developed by Rancher Labs, a software company that provides open-source tools and solutions for managing and deploying containers and Kubernetes.

Rancher is a complete software stack for teams adopting containers. It addresses the operational and security challenges of managing multiple Kubernetes clusters, while providing DevOps teams with integrated tools for running containerized workloads. Rancher provides centralized management, monitoring, and alerting for multiple clusters, regardless of where they are located, and it includes a rich set of features for automating the deployment and scaling of containerized applications.

K3s is a popular Kubernetes distribution that is often used with Rancher, as Rancher provides an easy-to-use interface for managing K3s clusters. Together, K3s and Rancher provide a lightweight and powerful solution for deploying and managing containerized workloads at scale.

## Rancher support matrix

The Rancher support matrix is a table that lists the versions of Kubernetes, Rancher, and other components that are compatible with each other. It is used to help users determine which versions of Rancher and Kubernetes they should use together, and which versions of other components are compatible with those versions.

The matrix is divided into three sections:

Rancher Server Support Matrix: This section lists the versions of Rancher that are compatible with each version of Kubernetes. It also lists the versions of Docker, Helm, and Istio that are supported with each version of Rancher.

Kubernetes Version Support Matrix: This section lists the versions of Kubernetes that are compatible with each version of Rancher. It also lists the versions of etcd, Calico, and other components that are supported with each version of Kubernetes.

Rancher Kubernetes Engine (RKE) Support Matrix: This section lists the versions of RKE that are compatible with each version of Kubernetes. It also lists the versions of Docker, etcd, and other components that are supported with each version of RKE.

The Rancher support matrix is regularly updated to reflect the latest versions and compatibility information. It is important to consult the matrix before deploying Rancher and Kubernetes, to ensure that you are using compatible versions of all components.

## Raspberry Pi configuration

Open `/boot/firmware/cmdline.txt`, add `cgroup_memory=1 cgroup_enable=memory` to the very end of the file:

```ini
console=serial0,115200 console=tty1 root=PARTUUID=96955a5f-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=VN cgroup_memory=1 cgroup_enable=memory
```

## Full script

### Install K3S, Helm and cert-manager

```bash
#!/bin/bash

# Set K3S_VERSION to the desired version
export INSTALL_K3S_VERSION=v1.26.12+k3s1
export DOMAIN="thanhtunguet.info"
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

# Get the K3S cluster token and server URL
export K3S_NODE_TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token)
echo "K3S_NODE_TOKEN=\"${K3S_NODE_TOKEN}\""

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0 \
  --set installCRDs=true
```

### Add Rancher repository (stable)

```bash
# Add Rancher repository
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

# Update your local Helm chart repository cache
helm repo update
```

### Install Rancher using LetsEncrypt

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

### Install Rancher using external SSL certificate

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
