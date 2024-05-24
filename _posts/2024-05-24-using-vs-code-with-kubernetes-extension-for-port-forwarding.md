---
title: "Using Visual Studio Code with Kubernetes Extension for Port Forwarding"
date: 2024-05-24 00:01:00 +0700
categories: [Devops, "Kubernetes"]
tags: ["Devops", "Kubernetes", "Rancher", "Visual Studio Code"]
pin: false
---

# Using Visual Studio Code with Kubernetes Extension for Port Forwarding

Visual Studio Code (VSCode) is a powerful and versatile code editor with a wide range of extensions to enhance your development workflow. One such extension is the Kubernetes extension, which allows you to manage Kubernetes clusters directly from VSCode. In this article, we'll explore how to use the Kubernetes extension to forward ports from Kubernetes pods to your local machine. Additionally, we'll cover how to add a new cluster to Kubernetes on VSCode by setting up the `kubeconfig` file, which can be obtained from Rancher.

## Prerequisites

Before we dive into the process, make sure you have the following prerequisites:

1. **Visual Studio Code**: Ensure you have VSCode installed on your machine. You can download it from [here](https://code.visualstudio.com/).

2. **Kubernetes Extension for VSCode**: Install the Kubernetes extension from the VSCode marketplace. You can find it [here](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools).

3. **Kubeconfig File**: Obtain your `kubeconfig` file from Rancher. This file contains the necessary configuration to connect to your Kubernetes cluster.

## Step-by-Step Guide

### Step 1: Installing the Kubernetes Extension

First, you need to install the Kubernetes extension for VSCode. Open VSCode and go to the Extensions view by clicking on the Extensions icon in the Activity Bar on the side of the window. Search for "Kubernetes" and click the Install button for the extension developed by Microsoft.

![alt text](</assets/img/Screenshot 2024-05-24 at 20.22.40.png>)


### Step 2: Obtaining the Kubeconfig File from Rancher

To connect to your Kubernetes cluster, you'll need the `kubeconfig` file from Rancher. Follow these steps to download or copy your `kubeconfig`:

1. Log in to Rancher.
2. Navigate to the cluster you want to connect to.
3. Click on the "Kubeconfig File" button to download the `kubeconfig` file. Alternatively, you can click "Copy to Clipboard" to copy the content of the `kubeconfig` file.

![alt text](</assets/img/Screenshot 2024-05-24 at 20.34.18.png>)

### Step 3: Adding a New Cluster to Kubernetes on VSCode

Now that you have your `kubeconfig` file, you can add your Kubernetes cluster to VSCode:

1. Open VSCode and go to the Command Palette by pressing `Ctrl+Shift+P` (or `Cmd+Shift+P` on macOS).
2. Type `Kubernetes: Add Existing Cluster...` and select it from the list.
3. You will be prompted to select the `kubeconfig` file. Choose the file you downloaded or copied from Rancher.
4. VSCode will now use the `kubeconfig` to connect to your Kubernetes cluster.

![alt text](</assets/img/Screenshot 2024-05-24 at 20.25.21.png>)

### Step 4: Forwarding Ports from Pods to Local Machine

With the cluster added, you can now forward ports from your Kubernetes pods to your local machine. Follow these steps:

1. Open the Kubernetes view in VSCode by clicking on the Kubernetes icon in the Activity Bar.
2. Expand your cluster and navigate to the "Workloads" section to see your pods.
3. Right-click on the pod you want to forward ports from and select `Port Forward`.
4. Specify the local port and the pod port you want to forward.

![alt text](</assets/img/Screenshot 2024-05-24 at 20.27.09.png>)

### Conclusion

By following these steps, you can easily manage your Kubernetes clusters and forward ports from your pods to your local machine using VSCode. The Kubernetes extension for VSCode streamlines these processes, making it a valuable tool for DevOps engineers and developers working with Kubernetes. Don't forget to add screenshots to the placeholders to visually guide your readers through each step.

Feel free to reach out if you have any questions or need further assistance. Happy coding!
