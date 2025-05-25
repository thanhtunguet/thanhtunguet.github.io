---
title: Using SSL on Kubernetes with cert-manager and Letsencrypt
date: 2023-01-29 19:10:00 +0700
categories: [Devops, Kubernetes]
tags: [Devops, Kubernetes, "Letsencrypt", "Cert Manager"]
pin: false
---

# Using SSL on Kubernetes with cert-manager and Letsencrypt

The basics of what HTTPS is and its benefits will not be covered in this article as this is quite basic knowledge and easy to look up. In this article, we will explore how to create HTTPS on Kubernetes.

## Ways to Create SSL

There are two popular ways here:

1. Buy single/wildcard certificates from agents

Pros:

- Simple/easy installation
- Long validity (up to 5 years)

Cons:

- Costs money, requires manual renewal when expired

2. Use Letsencrypt

Let's encrypt is [please visit their introduction link](https://letsencrypt.org/about/)

Simply put, Let's encrypt allows you to create free SSL, valid for up to 3 months.

Pros:

- Free
- Free
- Free

Important things are repeated three times.

- Can be automatically renewed using tools/jobs

Cons:

- Complex installation
- Short validity
- Requires setting up renewal jobs

## Why Kubernetes

Because we use Kubernetes, not why Kubernetes!

## What is Cert Manager?

On their website, they introduce:

```
X.509 certificate management for Kubernetes and OpenShift
```

Along with this image:

![https://cert-manager.io/images/cert-manager-graphic.svg](https://cert-manager.io/images/cert-manager-graphic.svg)

Basically, you install helm, then install the cert-manager chart, and you have an SSL management tool for you.

## Install cert-manager

Guide here: [https://cert-manager.io/docs/installation/helm/](https://cert-manager.io/docs/installation/helm/)

It's best to install via Helm, other methods ... up to you.

```bash
# If you have installed the CRDs manually instead of with the `--set installCRDs=true` option added to your Helm install command, you should upgrade your CRD resources before upgrading the Helm chart:
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1
```

## Domain Solvers

It is a way for CAs (Certificate authorities) to verify that you own the domain you are requesting SSL for. Why is it needed?
Ask the reverse: if this step didn't exist, anyone could verify and impersonate your domain.
Boom, no further explanation needed.

Google is still happy to sponsor.

To add the link: [https://cert-manager.io/docs/configuration/acme/](https://cert-manager.io/docs/configuration/acme/)

There are two main types of solvers (I don't know if there are others):

- HTTP
- DNS

Here I use Cloudflare's DNS.

### Why Cloudflare?

- Provides a very convenient API for DNS management
- Extremely fast update speed
- Supports wildcard domains

And more, but with these three reasons, we chose it :)

## How to Use?

### Create Issuer

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt # Issuer name
  namespace: default # Namespace to install the Issuer
spec:
  acme:
    email: admin@example.com # Your certificate management email
    privateKeySecretRef:
      name: prod-issuer-account-key
    server: https://acme-v02.api.letsencrypt.org/directory
    http01: {}
    solvers:
      - dns01:
          ingress:
            class: traefik # Ingress class, traefik or nginx
          cloudflare:
            apiTokenSecretRef:
              name: cloudflare-secret # Secret containing Cloudflare Token
              key: API_TOKEN # Token key
      - http01:
          ingress:
            class: traefik # Ingress class, traefik or nginx
          selector: {}
```

## Create Ingress with Certificates

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-name # Ingress name
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik # Ingress class, traefik or nginx
    cert-manager.io/issuer: letsencrypt # Issuer name created for the namespace
spec:
  issuerRef:
    name: letsencrypt
    namespace: default # Namespace you use, same namespace as the issuer
  tls:
    - secretName: subdomain-secret-tls # Secret containing TLS certificate
      hosts:
        - "subdomain.example.com" # Subdomain
  rules:
    - host: subdomain.example.com # Subdomain above
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-name # Your service
                port:
                  number: 80
```

Here you need to pay attention to a few things:

- `ingress class`: `traefik`, `nginx`
- `subdomain.example.com`: the subdomain you want to generate the corresponding certificate for.
  You can also use a wildcard domain
