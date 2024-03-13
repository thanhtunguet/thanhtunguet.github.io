---
title: Sử dụng SSL trên Kubernetes với cert-manager và Letsencrypt
date: 2023-01-29 19:10:00 +0700
categories: [Devops, Kubernetes]
tags: [Devops, Kubernetes, "Letsencrypt", "Cert Manager"]
pin: false
---

# Sử dụng SSL trên Kubernetes với cert-manager và Letsencrypt

Về cơ bản, HTTPS là gì và tác dụng của nó là gì, sẽ không được nhắc đến ở bài viết này do đây là kiến thức khá cơ bản và rất dễ để tra cứu. Trong bài này, ta sẽ tìm hiểu làm thế nào tạo được HTTPS trên Kubernetes.

## Các cách tạo SSL

Ở đây thì có 2 cách phổ biến

1. Mua single/wildcard certificates của các đại lý

Pros:

- Cài đặt đơn giản / dễ dàng
- Thời hạn sử dụng lâu (lên đến 5 năm)

Cons:

- Mất tiền, phải gia hạn bằng tay khi hết hạn

2. Sử dụng Letsencrypt

Let's encrypt là [mời bạn truy cập link họ tự giới thiệu](https://letsencrypt.org/about/)

Hiểu đơn giản, Let's encrypt cho phép bạn tạo SSL miễn phí, có thời hạn lên đến 3 tháng.

Pros:

- Miễn phí
- Miễn phí
- Miễn phí

Điều quan trọng nhắc lại 3 lần.

- Có thể tự động gia hạn bằng công cụ / jobs

Cons:

- Cài đặt phức tạp
- Thời hạn sử dụng ngắn
- Cần cài đặt job gia hạn

## Tại sao lại là Kubernetes

Bởi vì chúng ta dùng Kubernetes, chứ không phải tại sao lại là Kubernetes!

## Cert manager là gì?

Ở trang web của họ, họ giới thiệu:

```
X.509 certificate management for Kubernetes and OpenShift
```

Kèm theo cái hình này:

![https://cert-manager.io/images/cert-manager-graphic.svg](https://cert-manager.io/images/cert-manager-graphic.svg)

Về cơ bản, bạn cài helm, xong install cái chart cert-manager này, và xong bạn có một công cụ quản lý SSL cho bạn.

## Cài đặt cert-manager

Hướng dẫn tại đây: [https://cert-manager.io/docs/installation/helm/](https://cert-manager.io/docs/installation/helm/)

Tốt nhất là cài qua Helm, các cách khác ... tùy bạn.

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

Nó là cách để các CA (Certificate authority) xác thực bạn là chủ của cái domain bạn đang yêu cầu SSL. Tại sao cần nó ư?
Hỏi ngược lại nhé: nếu không có bước này thì ai cũng có thể xác thực và giả mạo domain của bạn.
Boom, không giải thích thêm nhé.

Google vẫn hân hạn tài trợ.

Để thêm cái link: [https://cert-manager.io/docs/configuration/acme/](https://cert-manager.io/docs/configuration/acme/)

Có 2 loại solver chính sau (tôi cũng chưa biết còn cái nào khác không):

- HTTP
- DNS

Thì ở đây tôi dùng DNS của Cloudflare.

### Tại sao lại là Cloudflare?

- Cung cấp API rất thuận tiện cho việc quản trị DNS
- Tốc độ cập nhật thay đổi cực nhanh
- Hỗ trợ wildcard domain

Còn nữa nhưng mà vì với 3 điều trên nên chúng tôi chọn nó :)

## Sử dụng thế nào?

### Tạo Issuer

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt # Tên Issuer
  namespace: default # Namespace cần cài đặt Issuer
spec:
  acme:
    email: admin@example.com # Email quản trị certificate của bạn
    privateKeySecretRef:
      name: prod-issuer-account-key
    server: https://acme-v02.api.letsencrypt.org/directory
    http01: {}
    solvers:
      - dns01:
          ingress:
            class: traefik # Ingress class, traefik hoặc nginx
          cloudflare:
            apiTokenSecretRef:
              name: cloudflare-secret # Secret chứa Cloudflare Token
              key: API_TOKEN # Key của token
      - http01:
          ingress:
            class: traefik # Ingress class, traefik hoặc nginx
          selector: {}
```

## Tạo Ingress with Certificates

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-name # Tên ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik # Ingress class, traefik hoặc nginx
    cert-manager.io/issuer: letsencrypt # Tên issuer đã tạo cho namespace
spec:
  issuerRef:
    name: letsencrypt
    namespace: default # Namespace bạn dùng, chung namespace với issuer
  tls:
    - secretName: subdomain-secret-tls # Secret chứa TLS certificate
      hosts:
        - "subdomain.example.com" # Subdomain
  rules:
    - host: subdomain.example.com # Subdomain ở trên
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-name # Service của bạn
                port:
                  number: 80
```

Ở đây bạn cần quan tâm một số thứ:

- `ingress class`: `traefik`, `nginx`
- `subdomain.example.com`: subdomain mà bạn muốn generate certificate tương ứng.
  Bạn cũng có thể sử dụng wildcard domain
