---
title: "How to Use SSH Port Forwarding to Access Your Server Behind NAT and Keep Connection Alive"
date: 2024-05-22 00:01:00 +0700
categories: ["DevOps", "Infrastructure"]
tags: ["DevOps", Linux]
pin: false
---

Accessing servers behind NAT (Network Address Translation) can be a challenge, especially when you need to ensure your connection remains alive. One powerful method to achieve this is using SSH (Secure Shell) port forwarding. This article will guide you through setting up SSH port forwarding and maintaining a persistent connection, with a practical script example for monitoring and restarting services if ports are closed, and scheduling this script to run every minute using cron.

## What is SSH Port Forwarding?

SSH port forwarding is a technique that allows you to securely tunnel network connections through an encrypted SSH connection. It enables you to access services on remote servers that are not directly accessible due to network restrictions like NAT.

## Prerequisites

1. **SSH access**: Ensure you have SSH access to the remote server.
2. **SSH client**: A machine with an SSH client installed (typically, Linux or macOS systems come with an SSH client by default).
3. **Basic knowledge of Linux**: Understanding basic Linux commands will help you follow along with the tutorial.

## Setting Up SSH Port Forwarding

### Step 1: Establish an SSH Connection with Port Forwarding

To set up port forwarding, you will use the `-L` option with the `ssh` command. Here’s the basic syntax:

```bash
ssh -L [local_port]:[remote_server]:[remote_port] user@remote_host
```

- `[local_port]`: The local port on your machine that will forward traffic.
- `[remote_server]`: The remote server’s address or hostname.
- `[remote_port]`: The port on the remote server to forward traffic to.
- `user@remote_host`: The username and hostname or IP address of the SSH server.

### Example Command

Assume you want to forward local port 8080 to remote server `example.com` port 62000:

```bash
ssh -L 8080:example.com:62000 user@example.com
```

This command creates a tunnel from your local machine’s port 8080 to port 62000 on `example.com`.

### Step 2: Keep the SSH Connection Alive

To ensure the SSH connection remains alive, you can use the `ServerAliveInterval` and `ServerAliveCountMax` options. These options send keep-alive messages at regular intervals.

Add these options to your SSH command:

```bash
ssh -L 8080:example.com:62000 user@example.com -o ServerAliveInterval=60 -o ServerAliveCountMax=5
```

- `ServerAliveInterval=60`: Sends a keep-alive message every 60 seconds.
- `ServerAliveCountMax=5`: If no response is received after 5 keep-alive messages, the connection is terminated.

## Monitoring and Restarting Services with a Bash Script

To ensure your services remain accessible, you can use a bash script to monitor the ports and restart the necessary services if they are closed. Here’s a script that does just that:

### The Script

```bash
#!/bin/bash

# Function to check if a port is open
check_port() {
  local address=$1
  local port=$2
  local timeout=5

  # Use telnet to check the port
  (echo > /dev/tcp/$address/$port) >/dev/null 2>&1
  return $?
}

# Function to restart a service
restart_service() {
  local service=$1
  echo "Restarting service $service..."
  sudo systemctl restart $service
}

# Define the address and ports
ADDRESS="example.com"
PORT1=62000
PORT2=62001

# Check port 62000
if ! check_port $ADDRESS $PORT1; then
  echo "Port $PORT1 is closed. Restarting ssh-service..."
  restart_service "ssh-service"
else
  echo "Port $PORT1 is open."
fi

# Check port 62001
if ! check_port $ADDRESS $PORT2; then
  echo "Port $PORT2 is closed. Restarting https-service..."
  restart_service "https-service"
else
  echo "Port $PORT2 is open."
fi
```

### Explanation

1. **Function `check_port`**: Checks if a specific port on a given address is open by attempting to establish a TCP connection.
2. **Function `restart_service`**: Restarts a given service using `systemctl`.
3. **Main Script**: Defines the address and ports, checks if they are open, and restarts the appropriate services if they are closed.

### Usage

1. Save the script to a file, for example, `check_ports.sh`.
2. Make the script executable:

    ```bash
    chmod +x check_ports.sh
    ```

3. Run the script:

    ```bash
    ./check_ports.sh
    ```

Ensure you have the necessary permissions to restart services (`sudo` may be required).

## Automating the Script with Cron

To run the script automatically every minute, you can add it to your cron jobs. Here’s how to do it:

### Step 1: Open the Crontab Editor

Open the crontab editor by running:

```bash
crontab -e
```

### Step 2: Add the Cron Job

Add the following line to schedule the script to run every minute:

```bash
* * * * * /path/to/check_ports.sh >> /path/to/check_ports.log 2>&1
```

Replace `/path/to/check_ports.sh` with the actual path to your script and `/path/to/check_ports.log` with the path where you want to save the log file.

### Step 3: Save and Exit

Save the file and exit the editor. The script will now run every minute, checking the specified ports and restarting the services if necessary.

## Conclusion

Using SSH port forwarding is a powerful way to access servers behind NAT, and keeping your connection alive ensures reliable access. With the provided bash script, you can monitor and restart services as needed, ensuring continuous availability. By adding the script to cron, you automate the process, maintaining stable connections and ensuring your services remain accessible. Follow these steps to effectively manage remote servers and maintain persistent connections.
