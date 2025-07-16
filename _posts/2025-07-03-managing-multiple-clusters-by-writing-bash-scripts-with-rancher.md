---
title: "Managing Multiple Kubernetes Clusters with Smart Bash Scripts: A DevOps Engineer's Toolkit"
date: 2025-07-03 07:50:00 +0700
categories: ["DevOps", "Containers"]
tags: [Devops, Kubernetes, Rancher]
pin: false
---

As a DevOps engineer working with multiple Kubernetes clusters, you've likely experienced the frustration of constantly switching between different environments, exporting kubeconfigs, and managing deployments across various namespaces. While tools like Rancher provide excellent cluster management capabilities, the daily operational tasks can still be time-consuming and error-prone when done manually.

Today, I'll share three powerful bash scripts that have transformed my multi-cluster workflow: `kexport`, `ksw`, and `kscale`. These tools demonstrate how thoughtful automation can dramatically improve your Kubernetes operations efficiency.

## The Challenge of Multi-Cluster Management

Before diving into the solutions, let's acknowledge the common pain points:

- **Context Switching Overhead**: Constantly running `kubectl config use-context` and managing multiple kubeconfig files
- **Configuration Management**: Keeping track of different cluster configurations and ensuring you're working with the right environment
- **Bulk Operations**: Scaling multiple deployments across namespaces during maintenance windows or load testing
- **Human Error**: Accidentally running commands against the wrong cluster or environment

## Solution 1: Automated Kubeconfig Export with `kexport`

The first script addresses the tedious task of manually exporting kubeconfig files for each cluster managed by Rancher.

```bash
#!/bin/zsh

CONFIG_DIR="$HOME/.kube"
mkdir -p "$CONFIG_DIR"

# Get all cluster names from Rancher CLI output
CLUSTERS=$(rancher clusters | awk 'NR>1 {print $3}' | sed 's/*//g')

if [[ -z "$CLUSTERS" ]]; then
    echo "No clusters found in Rancher. Ensure you are logged in."
    exit 1
fi

# Loop through each cluster and export its kubeconfig
for CLUSTER in ${(f)CLUSTERS}; do
    if [[ "$CLUSTER" != "active" ]]; then
        CONFIG_FILE="$CONFIG_DIR/$CLUSTER.yaml"
        echo "Exporting kubeconfig for cluster: $CLUSTER"
        rancher clusters kf "$CLUSTER" --export > "$CONFIG_FILE"
        echo "Saved: $CONFIG_FILE"
    fi
done

echo "All cluster kubeconfigs have been exported successfully."
```

**Key Features:**
- **Automated Discovery**: Parses Rancher CLI output to identify all available clusters
- **Batch Processing**: Exports all cluster configurations in a single command
- **Error Handling**: Validates Rancher authentication and filters out invalid entries
- **Organized Storage**: Saves configurations with descriptive names in a standard location

**Usage**: Simply run `kexport` after logging into Rancher, and all your cluster kubeconfigs will be ready for use.

## Solution 2: Seamless Cluster Switching with `ksw`

The second script revolutionizes how you switch between clusters by providing both interactive and direct selection methods.

```bash
#!/bin/zsh

CONFIG_DIR="$HOME/.kube"

# Interactive selection if no argument provided
if [ -z "$1" ]; then
    CLUSTERS=($(find "$CONFIG_DIR" -name "*.yaml" -type f -exec basename {} .yaml \;))
    if [ ${#CLUSTERS[@]} -eq 0 ]; then
        echo "No kubeconfig files found in $CONFIG_DIR"
        exit 1
    fi
    echo "Select a cluster to switch to:"
    select CLUSTER in "${CLUSTERS[@]}"; do
        if [ -n "$CLUSTER" ]; then
            export KUBECONFIG="$CONFIG_DIR/$CLUSTER.yaml"
            cat "$CONFIG_DIR/$CLUSTER.yaml" > ~/.kube/config
            exit 0
        else
            echo "Invalid selection, please try again."
        fi
    done
    exit 0
fi

# Direct cluster selection
CONFIG_FILE="$CONFIG_DIR/$1.yaml"
if [ -f "$CONFIG_FILE" ]; then
    export KUBECONFIG="$CONFIG_FILE"
    echo "export KUBECONFIG=\"$CONFIG_FILE\""
    cat "$CONFIG_FILE" > ~/.kube/config
else
    echo "Error: Kubeconfig for '$1' not found in $CONFIG_DIR"
    exit 1
fi
```

**Key Features:**
- **Dual Interface**: Interactive menu for exploration, direct argument for automation
- **Safe Switching**: Validates cluster existence before switching
- **Standard Integration**: Updates the default kubeconfig location for seamless kubectl usage
- **Visual Feedback**: Clear confirmation of successful switches

**Usage Examples:**
- `ksw` - Interactive cluster selection menu
- `ksw production` - Direct switch to production cluster
- `ksw staging` - Direct switch to staging cluster

## Solution 3: Bulk Deployment Scaling with `kscale`

The third script addresses the common need to scale multiple deployments simultaneously, particularly useful during maintenance windows or load testing scenarios.

```bash
#!/bin/bash

NAMESPACE="$1"
REPLICAS="$2"

# Get all deployments in the specified namespace
DEPLOYMENTS=$(kubectl get deployments -n $NAMESPACE -o custom-columns=NAME:.metadata.name --no-headers)

# Scale each deployment to the specified replica count
for deployment in $DEPLOYMENTS; do
    kubectl scale --replicas="${REPLICAS}" deployment/$deployment -n $NAMESPACE
    echo "Scaled deployment $deployment to $REPLICAS pods."
done

echo "All deployments scaled to $REPLICAS pods in namespace $NAMESPACE."
```

**Key Features:**
- **Namespace-Wide Operations**: Targets all deployments within a specified namespace
- **Flexible Scaling**: Supports scaling up or down to any replica count
- **Progress Tracking**: Provides real-time feedback on scaling operations
- **Simplicity**: Clean, focused functionality without unnecessary complexity

**Usage Examples:**
- `kscale production 0` - Scale down all production deployments for maintenance
- `kscale staging 3` - Scale up all staging deployments for load testing
- `kscale development 1` - Reset development environment to minimal resources

## Best Practices and Considerations

When implementing similar automation scripts, consider these important practices:

**Security Considerations:**
- Always validate cluster context before destructive operations
- Consider implementing confirmation prompts for production environments
- Use appropriate file permissions for kubeconfig files (600 or 644)

**Error Handling:**
- Implement robust error checking for external command dependencies
- Provide meaningful error messages for troubleshooting
- Gracefully handle network timeouts and authentication failures

**Extensibility:**
- Design scripts to be easily modified for different environments
- Consider configuration files for frequently changed parameters
- Document assumptions about external tool versions and configurations

**Integration:**
- Make scripts compatible with common shell environments
- Consider adding shell completion for frequently used parameters
- Integrate with existing CI/CD workflows where appropriate

## Conclusion

These three scripts demonstrate how targeted automation can significantly improve your multi-cluster Kubernetes workflow. By investing time in creating these tools, you'll save countless hours of manual work while reducing the risk of human error in critical operations.

The key to successful DevOps automation lies in identifying repetitive tasks and creating focused solutions that solve specific problems well. Rather than building monolithic tools, these scripts follow the Unix philosophy of doing one thing exceptionally well while being composable with other tools.

Start by implementing these scripts in your own environment, then customize them to match your specific workflow requirements. Remember that the best automation tools are those that eliminate friction from your daily operations while maintaining the flexibility to handle edge cases.

Whether you're managing a handful of clusters or dozens of environments, these patterns will help you build a more efficient and reliable operational toolkit. The investment in script development pays dividends in reduced operational overhead and increased confidence in your deployment processes.