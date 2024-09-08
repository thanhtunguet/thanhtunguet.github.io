---
title: "How to Completely Remove Rancher from a Kubernetes Cluster"
date: 2024-09-08 12:45:00 +0700
categories: ["Devops", "Kubernetes"]
tags: ["Devops", "Kubernetes", "Linux", "Rancher"]
pin: false
mermaid: true
---

### How to Completely Remove Rancher from a Kubernetes Cluster

If you need to fully remove Rancher from your Kubernetes cluster, this guide will walk you through the process step-by-step. We will cover how to delete all resources related to Rancher, remove webhook configurations, and handle namespaces that may get stuck in a terminating state.

#### Step 1: Identify Rancher-Related Namespaces

Rancher typically installs several namespaces in your Kubernetes cluster:

- `cattle-system`: Contains core Rancher components and services.
- `cattle-impersonation-system`: Handles impersonation for access management.
- `cattle-fleet-system`: Related to Fleet, Rancher's GitOps tool.

To begin, list all the namespaces in your cluster and identify the Rancher-related namespaces:

```bash
kubectl get namespaces
```

#### Step 2: Delete All Resources in Rancher-Related Namespaces

You need to delete all resources in the Rancher-related namespaces. Run the following commands for each namespace:

```bash
# Define namespaces to delete resources from
NAMESPACES=("cattle-system" "cattle-impersonation-system" "cattle-fleet-system")

# Loop through each namespace and delete all resources
for NAMESPACE in "${NAMESPACES[@]}"; do
  echo "Deleting all resources in namespace: $NAMESPACE"
  kubectl delete all --all -n $NAMESPACE
done
```

#### Step 3: Remove Rancher Webhook Configurations

Rancher installs validating and mutating webhook configurations to enforce policies in your cluster. To remove these configurations:

1. **List and Delete Validating Webhook Configurations:**

   ```bash
   kubectl get validatingwebhookconfigurations
   kubectl delete validatingwebhookconfiguration rancher.cattle.io
   ```

2. **List and Delete Mutating Webhook Configurations:**

   ```bash
   kubectl get mutatingwebhookconfigurations
   kubectl delete mutatingwebhookconfiguration rancher.cattle.io
   ```

Replace `rancher.cattle.io` with the actual names if they differ.

#### Step 4: Delete Rancher-Related Namespaces

After removing the resources and webhooks, delete the Rancher-related namespaces:

```bash
for NAMESPACE in "${NAMESPACES[@]}"; do
  echo "Deleting namespace: $NAMESPACE"
  kubectl delete namespace $NAMESPACE
done
```

#### Step 5: Handling Stuck Namespaces (Terminating State)

Sometimes, namespaces can get stuck in a terminating state due to finalizers blocking the deletion. To forcefully delete such namespaces, follow these steps:

1. **Identify Stuck Namespaces:**

   Check if any namespaces are stuck in the terminating state:

   ```bash
   kubectl get namespaces
   ```

2. **Remove Finalizers to Force Deletion:**

   Replace `your-namespace` with the stuck namespace name. Run the following commands:

   ```bash
   NAMESPACE="your-namespace"

   # Get the namespace JSON and remove finalizers
   kubectl get namespace $NAMESPACE -o json | jq '.spec.finalizers=[]' > temp-namespace.json

   # Apply the modified namespace configuration
   kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f temp-namespace.json

   # Clean up temporary file
   rm temp-namespace.json
   ```

This command retrieves the namespace's JSON definition, removes the `finalizers` field, and then replaces the existing configuration to forcefully delete the namespace.

#### Step 6: Verify Cleanup

After deleting all resources, namespaces, and webhook configurations, verify that no Rancher-related components remain in your cluster:

1. **Check for Remaining Namespaces:**

   ```bash
   kubectl get namespaces
   ```

2. **Check for Remaining Webhooks:**

   ```bash
   kubectl get validatingwebhookconfigurations
   kubectl get mutatingwebhookconfigurations
   ```

Ensure there are no Rancher-related namespaces or webhook configurations left in your cluster.

### Full Script for Complete Removal

Below is a full script that automates the process of removing all Rancher-related resources, webhook configurations, and namespaces from your Kubernetes cluster. This script also handles namespaces that are stuck in the terminating state by overriding their finalizers.

```bash
#!/bin/bash

# Define Rancher-related namespaces
NAMESPACES=("cattle-system" "cattle-impersonation-system" "cattle-fleet-system")

# Delete all resources in each namespace
for NAMESPACE in "${NAMESPACES[@]}"; do
  echo "Deleting all resources in namespace: $NAMESPACE"
  kubectl delete all --all -n $NAMESPACE
done

# Remove Rancher webhook configurations
echo "Removing Rancher webhook configurations..."
kubectl get validatingwebhookconfigurations | grep 'rancher' | awk '{print $1}' | xargs kubectl delete validatingwebhookconfiguration
kubectl get mutatingwebhookconfigurations | grep 'rancher' | awk '{print $1}' | xargs kubectl delete mutatingwebhookconfiguration

# Delete Rancher-related namespaces
for NAMESPACE in "${NAMESPACES[@]}"; do
  echo "Deleting namespace: $NAMESPACE"
  kubectl delete namespace $NAMESPACE
done

# Handle stuck namespaces by removing finalizers
for NAMESPACE in "${NAMESPACES[@]}"; do
  if kubectl get namespace $NAMESPACE -o jsonpath='{.status.phase}' | grep -q 'Terminating'; then
    echo "Namespace $NAMESPACE is stuck in Terminating state. Removing finalizers..."
    kubectl get namespace $NAMESPACE -o json | jq '.spec.finalizers=[]' > temp-$NAMESPACE.json
    kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f temp-$NAMESPACE.json
    rm temp-$NAMESPACE.json
  fi
done

echo "Rancher removal process is complete. Verify the cleanup to ensure no Rancher-related resources remain."
```

#### Conclusion

By following these steps and using the provided script, you can successfully remove Rancher and all its components from your Kubernetes cluster. This process ensures a clean removal, eliminating potential conflicts or issues from remnants of the Rancher installation.

Remember to review your cluster and verify that no unexpected changes occurred during this removal process. If you plan to reinstall Rancher or migrate to a different cluster management tool, consider taking necessary backups and precautions before making significant changes.
