---
title: "Automating Kubernetes Workload Backups with Bash and Git"
date: 2024-02-04 07:45:00 +0700
categories: [Devops, Kubernetes]
tags: [Devops, Kubernetes, Linux, Scripts]
pin: false
---

### Introduction

Kubernetes has revolutionized container orchestration, offering powerful tools for deploying and managing containerized applications. However, ensuring the resilience and recoverability of your Kubernetes workloads is equally critical. In this article, we'll dive into a common challenge faced by DevOps engineers and system administrators: automating the backup of Kubernetes workloads. We'll introduce a Bash script that uses the `kubectl` command and Git to provide a practical solution to this problem.

### The Challenge:

Kubernetes simplifies the management of container workloads, but maintaining a reliable backup of your configurations, including services, deployments, configmaps, ingress rules, and secrets, can be challenging. Here are some common issues:

1.  **Resource Diversity**: Kubernetes supports various resource types, each with its own configuration details. Manually backing up these diverse resources can be time-consuming and prone to errors.

2.  **Namespace Complexity**: Workloads are often organized into namespaces for better isolation and resource management. Managing backups across multiple namespaces can become complex.

3.  **Version Control**: Storing backups in an organized and version-controlled manner is crucial for traceability and disaster recovery. Git is an excellent choice for version control, but integrating it into the backup process requires a tailored solution.

### The Solution: A Bash Script

To address these challenges, we've created a Bash script that streamlines the backup process for Kubernetes workloads. This script uses the `kubectl` command-line tool, Git for version control, and Bash scripting to automate the backup of Kubernetes resources.

### Script Overview:

Let's break down the Bash script step by step:

```bash
#!/bin/bash
```

This line specifies that the script should be executed using the Bash shell.

```bash
cd "/home/k8s/kubebackup"
```

Here, we change the current working directory to "/home/k8s/kubebackup" to ensure that the script operates within the desired directory.

```bash
function usage() {
    echo "Usage: $0 [options]"
    echo "Options:"
    echo "  --type, -t          Type of resource: service|deployment|configmaps|ingress|secret"
    echo "  --namespace, -n     Namespace"
    echo "  --output, -o        Output directory"
    exit 1
}
```

We define a `usage()` function that provides information about how to use the script, including its options. This function is used to display usage instructions when incorrect arguments are provided.

```bash
# Initialize variables
backup_dir="k8s-backup-$(date +'%Y%m%d%H%M%S')"
```

We initialize the `backup_dir` variable with a default backup directory name that includes a timestamp to make it unique.

```bash
# Command line argument parsing
# ...
```

The script uses a `while` loop and a `case` statement to parse command-line arguments. It checks for options like `--type`, `--namespace`, and `--output`, assigning values to corresponding variables (`resource_type`, `namespace`, and `backup_dir`) as needed.

```bash
# Create backup directory
mkdir -p "$backup_dir"
```

This line creates the backup directory specified by the `backup_dir` variable. The `-p` flag ensures that the directory and its parent directories are created if they don't exist.

```bash
# Backup resources function
# ...
```

The core functionality of the script is encapsulated within the `backup_resources()` function, which we'll explore in detail shortly.

```bash
# Execute backup process based on provided options
# ...
```

This section of the script determines whether a specific namespace and resource type are provided. It then initiates the backup process accordingly, either for a specific resource type in a given namespace or for all supported resource types in that namespace.

```bash
# Git integration
git add -A
git commit -m "Backup ($(date +'%Y-%m-%d %H:%M:%S'))"
git push origin main
```

After completing the backup operations, the script adds all changes to a Git repository, commits them with a message that includes a timestamp, and pushes the changes to a remote repository. This ensures that your Kubernetes configurations are version-controlled and easily restorable in case of issues.

### The full script

Below is the full script of the backup process

```bash
#!/bin/bash

cd "/home/k8s/kubebackup"

function usage() {
    echo "Usage: $0 [options]"
    echo "Options:"
    echo "  --type, -t          Type of resource: service|deployment|configmaps|ingress|secret"
    echo "  --namespace, -n     Namespace"
    echo "  --output, -o        Output directory"
    exit 1
}

backup_dir="k8s-backup-$(date +'%Y%m%d%H%M%S')"

while [[ $# -gt 0 ]]; do
    case "$1" in
        --type|-t)
            resource_type="$2"
            shift 2
        ;;
        --namespace|-n)
            namespace="$2"
            shift 2
        ;;
        --output|-o)
            backup_dir="$2"
            shift 2
        ;;
        *)
            echo "Invalid option: $1"
            usage
        ;;
    esac
done

mkdir -p "$backup_dir"

function backup_resources() {
    local resource_type="$1"
    local namespace="$2"
    local resource_plural=""
    
    case "$resource_type" in
        service)
            resource_plural="services"
        ;;
        deployment)
            resource_plural="deployments"
        ;;
        configmaps)
            resource_plural="configmaps"
        ;;
        ingress)
            resource_plural="ingresses"
        ;;
        secret)
            resource_plural="secrets"
        ;;
        *)
            echo "Unsupported resource type: $resource_type"
            exit 1
        ;;
    esac
    
    local namespace_backup_dir="$backup_dir/$namespace"
    mkdir -p "$namespace_backup_dir"
    
    kubectl get "$resource_plural" -n "$namespace" -o yaml > "$namespace_backup_dir/${resource_type}s.yaml"
    
    echo "Backup for $resource_type in namespace $namespace completed"
}

if [[ -n "$namespace" ]]; then
    user_namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n')
    
    if ! echo "$user_namespaces" | grep -q "$namespace"; then
        echo "Namespace $namespace does not exist or is not a user namespace"
        exit 1
    fi
    
    if [[ -n "$resource_type" ]]; then
        backup_resources "$resource_type" "$namespace"
    else
        backup_resources "service" "$namespace"
        backup_resources "deployment" "$namespace"
        backup_resources "configmaps" "$namespace"
        backup_resources "ingress" "$namespace"
        backup_resources "secret" "$namespace"
    fi
    
    echo "Backup process for namespace $namespace completed"
else
    user_namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n')
    
    for namespace in $user_namespaces; do
        if [[ -n "$resource_type" ]]; then
            backup_resources "$resource_type" "$namespace"
        else
            backup_resources "service" "$namespace"
            backup_resources "deployment" "$namespace"
            backup_resources "configmaps" "$namespace"
            backup_resources "ingress" "$namespace"
            backup_resources "secret" "$namespace"
        fi
    done
    
    echo "Backup process completed"
fi

git add -A
git commit -m "Backup ($(date +'%Y-%m-%d %H:%M:%S'))"
git push origin main
```

### Benefits of the Solution:

Now that we understand how the script works, let's highlight the benefits it offers:

1.  **Automation**: The script automates the resource backup process, reducing manual effort and the potential for human error.

2.  **Flexibility**: Users can choose specific resource types and namespaces to back up, allowing for customization based on their Kubernetes deployment.

3.  **Version Control**: By integrating Git, the script ensures that backups are organized, versioned, and easily restorable.

4.  **Consistency**: The script enforces a consistent backup process across different resource types and namespaces.

5.  **Traceability**: The use of Git enables traceability of changes to Kubernetes configurations over time.

### Conclusion:

Backing up Kubernetes workloads is essential for maintaining the integrity and recoverability of your containerized applications. Our Bash script offers a practical solution by automating the backup process, allowing you to specify resource types and namespaces, and seamlessly integrating Git for version control. By using this script, you can ensure that your Kubernetes configurations are consistently backed up and versioned, making disaster recovery and troubleshooting more manageable tasks in your DevOps workflow.

With this tool in your arsenal, you can focus on deploying and scaling your containerized applications with confidence, knowing that you have a robust backup strategy in place to safeguard your Kubernetes workloads.
