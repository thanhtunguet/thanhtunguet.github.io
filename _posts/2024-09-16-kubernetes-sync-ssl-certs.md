---
title: Automating Certificate Sync Between Ubuntu and Kubernetes Clusters
date: 2024-09-16 14:10:00 +0700
categories: ["DevOps", "Containers"]
tags: [Devops, Kubernetes, "Letsencrypt", "Cert Manager"]
pin: false
---

Managing SSL/TLS certificates can be a challenging task, especially when you have multiple domains and need to ensure that all your Kubernetes clusters are up-to-date with the latest certificates. In this blog post, we'll explore an efficient way to automate syncing Let's Encrypt certificates from an Ubuntu server to multiple Kubernetes clusters using a simple bash script. We'll go through the problem, provide a script to solve it, and explain how this solution works.

### **The Problem: Keeping Certificates in Sync Across Multiple Clusters**

When using Let's Encrypt certificates for multiple domains, especially wildcard certificates, it's common to run Certbot on an Ubuntu server to renew them. However, managing these certificates becomes more complex when they need to be synchronized across multiple Kubernetes clusters in various namespaces.

Certbot automatically renews the certificates, but these updated certificates need to be distributed to all relevant Kubernetes clusters promptly. Manually copying the certificates and updating Kubernetes secrets can be error-prone and time-consuming, especially if you manage multiple clusters. Automating this process ensures consistency, saves time, and minimizes the risk of downtime due to expired certificates.

### **The Solution: A Bash Script to Sync Certificates**

To automate this process, we can create a bash script (`sync-certificate.sh`) that will:

1. Retrieve the renewed certificates from the Ubuntu machine.
2. Encode the certificate and private key in base64 format, as required by Kubernetes secrets.
3. Create or update the secrets in the specified namespaces across multiple Kubernetes clusters using the provided kubeconfig.

The script will take care of everything, from fetching the certificates to applying them to the clusters.

### **The Bash Script: `sync-certificate.sh`**

Here's the bash script that automates the synchronization process:

```bash
#!/bin/bash

# Function to display usage
usage() {
  echo "Usage: $0 --kubeconfig /path/to/kube/config.yaml --namespace namespace --domain example.domain"
  exit 1
}

# Check if correct number of arguments is provided
if [ "$#" -ne 6 ]; then
  usage
fi

# Parse command-line arguments
while [[ "$#" -gt 0 ]]; do
  case $1 in
    --kubeconfig) KUBECONFIG="$2"; shift ;;
    --namespace) NAMESPACE="$2"; shift ;;
    --domain) DOMAIN="$2"; shift ;;
    *) usage ;;
  esac
  shift
done

# Verify that required arguments are provided
if [ -z "$KUBECONFIG" ] || [ -z "$NAMESPACE" ] || [ -z "$DOMAIN" ]; then
  usage
fi

# Certbot certificates directory
CERT_DIR="/etc/letsencrypt/live/$DOMAIN"

# Check if the certificate directory exists
if [ ! -d "$CERT_DIR" ]; then
  echo "Certificate directory $CERT_DIR does not exist. Please check the domain name."
  exit 1
fi

# Get the certificate files
TLS_CERT=$(cat "$CERT_DIR/fullchain.pem")
TLS_KEY=$(cat "$CERT_DIR/privkey.pem")

# Create a temporary Kubernetes secret YAML file
SECRET_YAML=$(mktemp)
cat <<EOF > "$SECRET_YAML"
apiVersion: v1
kind: Secret
metadata:
  name: $DOMAIN
  namespace: $NAMESPACE
type: kubernetes.io/tls
data:
  tls.crt: $(echo "$TLS_CERT" | base64 | tr -d '\n')
  tls.key: $(echo "$TLS_KEY" | base64 | tr -d '\n')
EOF

# Check if the secret already exists
if KUBECONFIG="$KUBECONFIG" kubectl get secret "$DOMAIN" -n "$NAMESPACE" > /dev/null 2>&1; then
  echo "Secret $DOMAIN already exists in namespace $NAMESPACE. Deleting it..."
  KUBECONFIG="$KUBECONFIG" kubectl delete secret "$DOMAIN" -n "$NAMESPACE"
fi

# Apply the secret to the Kubernetes cluster
echo "Creating new secret $DOMAIN in namespace $NAMESPACE..."
KUBECONFIG="$KUBECONFIG" kubectl apply -f "$SECRET_YAML" --kubeconfig="$KUBECONFIG"

# Cleanup temporary file
rm -f "$SECRET_YAML"

echo "Certificate for $DOMAIN has been synced to namespace $NAMESPACE using kubeconfig $KUBECONFIG."
```

### **How the Script Solves the Issue**

#### **1. Automates the Certificate Sync Process**

By using this script, you automate the entire process of fetching certificates from the Let's Encrypt directory (`/etc/letsencrypt/live/<domain>`), encoding them in base64, and updating or creating the Kubernetes secrets across multiple clusters and namespaces. This automation eliminates the need for manual intervention, reducing errors and saving time.

#### **2. Ensures Certificate Updates Are Applied**

The script first checks if a secret for the specified domain already exists in the target namespace. If it does, the script deletes the existing secret to ensure a clean update with the latest certificates. This step guarantees that the certificates in your Kubernetes clusters are always up-to-date.

#### **3. Supports Multiple Clusters and Namespaces**

By allowing you to specify the kubeconfig file, namespace, and domain, this script can be easily adapted to sync certificates across multiple clusters and namespaces. You can run the script multiple times with different arguments to handle all your clusters.

#### **4. Easy to Integrate with Cron Jobs for Regular Updates**

You can set up a cron job on your Ubuntu machine to run this script periodically, ensuring that your Kubernetes clusters are always updated with the latest certificates. For example, to run the script every night at midnight, you can add the following line to your crontab:

```sh
0 0 * * * /path/to/sync-certificate.sh --kubeconfig /path/to/kube/config.yaml --namespace your-namespace --domain example.com >> /var/log/sync-certificates.log 2>&1
```

### **Conclusion**

Keeping your SSL/TLS certificates in sync across multiple Kubernetes clusters can be a daunting task, but with this bash script, the process becomes seamless and automated. By setting up a cron job to run this script regularly, you ensure that all your clusters are always up-to-date with the latest certificates, minimizing downtime and maintaining security.

By automating certificate management, you save time, reduce errors, and ensure consistency across your infrastructure. Try out this script and make your certificate management hassle-free! 

If you have any questions or want to share your experiences, feel free to leave a comment below. Happy automating!
