---
title: "Adding a New Node to a Kubernetes Cluster Managed by KubeSphere"
date: 2024-12-08 12:30:00 +0700
categories: [DevOps, Kubernetes]
categories: ["DevOps", "Containers"]
pin: false
mermaid: false
---

### **Introduction**

KubeSphere is an elegant, easy-to-use Kubernetes platform that simplifies cluster management and adds enterprise-level features. Adding a new node to an existing Kubernetes cluster managed by KubeSphere is a common operation to expand resources or improve fault tolerance. This guide provides a step-by-step walkthrough for adding a node to your cluster and addresses common issues faced during the process.

---

### **Prerequisites**

1. **Existing Kubernetes Cluster**:
   - A running Kubernetes cluster (e.g., version `1.26.10`) managed by KubeSphere.

2. **New Node Preparation**:
   - A machine (physical or virtual) with a supported OS (e.g., Ubuntu 20.04 or CentOS 7/8).
   - Basic software installed: `ssh`, `curl`, and `iptables`.

3. **Access to the Control Plane**:
   - SSH access to the control plane node where KubeKey or the cluster configuration resides.

4. **Cluster Configuration File**:
   - The `cluster.yaml` file used to configure the cluster during its setup.

5. **Permissions**:
   - You can work as a non-root user with `sudo` privileges, though `root` access is preferred for simplicity.

---

### **Step 1: Verify Cluster and KubeKey Installation**

1. **Check the Current Cluster Status**:
   Run the following commands on the control plane node:
   ```bash
   kubectl get nodes
   kubectl get pods -A
   ```

   This confirms the health of your existing cluster.

2. **Install KubeKey (if not already installed)**:
   Download and install the KubeKey binary:
   ```bash
   curl -sfL https://get-kk.kubesphere.io | sudo bash
   ```
   Verify the installation:
   ```bash
   kk version
   ```

---

### **Step 2: Retrieve or Recreate `cluster.yaml`**

1. **Locate the Existing Configuration**:
   - If the `cluster.yaml` file exists, locate it in the working directory used during the initial cluster setup.

2. **Regenerate `cluster.yaml` (if lost)**:
   Use the `kubectl` tool and `kk` to generate a new configuration file based on the current cluster:
   ```bash
   kk create config --from-cluster
   ```
   Save the file as `cluster.yaml`.

---

### **Step 3: Add Node Information to `cluster.yaml`**

Edit the `cluster.yaml` file to include the new node. Here's an example section for adding a worker node:

```yaml
hosts:
  - {name: "worker-node-2", address: "192.168.1.102", internalAddress: "192.168.1.102", user: "ubuntu", password: "your-password"}
```

> Replace `192.168.1.102`, `ubuntu`, and `your-password` with the new node's IP address, SSH username, and password.

If you're using SSH keys instead of passwords:
```yaml
  - {name: "worker-node-2", address: "192.168.1.102", internalAddress: "192.168.1.102", user: "ubuntu", privateKeyPath: "~/.ssh/id_rsa"}
```

---

### **Step 4: Add the Node to the Cluster**

1. **Run the KubeKey Command**:
   Execute the following command to apply the changes and add the new node:
   ```bash
   kk add nodes -f cluster.yaml
   ```

   KubeKey will:
   - Install Kubernetes and required components on the new node.
   - Configure the node as a part of the cluster.

2. **Monitor the Progress**:
   The process may take several minutes. Monitor the output for errors or warnings.

---

### **Step 5: Verify the New Node**

1. **Check Node Status**:
   Run the following command to ensure the new node is added and ready:
   ```bash
   kubectl get nodes
   ```

   You should see the new node listed with a `Ready` status.

2. **Verify Workload Distribution**:
   Check if pods are being scheduled on the new node:
   ```bash
   kubectl get pods -o wide -A
   ```

---

### **Step 6: (Optional) Enable KubeSphere Features**

If the new node requires specific KubeSphere components or services:
1. Log in to the KubeSphere web console.
2. Navigate to **Cluster Management** > **Nodes**.
3. Assign roles or workloads to the new node as needed.

---

### **Common Issues and Fixes**

#### **Issue 1: SSH Connection Fails**
- **Cause**: Incorrect SSH credentials or configuration.
- **Solution**:
  - Ensure SSH is enabled on the new node.
  - Test the connection manually:
    ```bash
    ssh ubuntu@192.168.1.102
    ```

#### **Issue 2: Node Stuck in `NotReady` State**
- **Cause**: Missing dependencies or incorrect configuration.
- **Solution**:
  - Check the node logs:
    ```bash
    kubectl describe node worker-node-2
    journalctl -u kubelet
    ```
  - Verify network and container runtime setup on the new node.

#### **Issue 3: API Server Connection Refused**
- **Cause**: The new node cannot communicate with the control plane.
- **Solution**:
  - Verify that the `apiserver` address is correctly set in `cluster.yaml`.
  - Ensure firewall and network policies allow communication.

---

### **Best Practices**

1. **Plan Node Names and Roles**:
   Use clear naming conventions for nodes to identify their purpose and roles.

2. **Backup Configuration**:
   Always back up your `cluster.yaml` and KubeSphere configuration before making changes.

3. **Monitor Cluster Health**:
   Use tools like `kubectl top` and KubeSphere’s monitoring features to track resource usage and node performance.

4. **Test Node Addition in Staging**:
   Test the node addition process in a non-production environment to avoid service disruptions.

---

### **Conclusion**

Expanding a Kubernetes cluster with KubeSphere is a straightforward process when following the correct steps. By carefully configuring and monitoring the addition of new nodes, you can ensure a smooth scaling operation without impacting the cluster’s stability. KubeSphere's user-friendly tools further simplify this task, making it accessible for teams of all sizes.
