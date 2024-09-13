---
title: "Creating port-forwarding service for Linux to expose your server to Internet"
date: 2024-02-04 07:45:00 +0700
categories: [Devops, Linux]
tags: [Devops, SSH, Linux, Scripts]
pin: false
---

In today's interconnected world, securely exposing local server ports to the internet is a common need, especially when dealing with setups where a local server resides behind a NAT (Network Address Translation) and a public VPS (Virtual Private Server) with its own public IP is available. One of the most effective and secure methods to achieve this is through SSH port forwarding. In this guide, we'll walk through the steps to configure a Linux service to accomplish this task, ensuring that it automatically restarts every minute in case of failure.

## Understanding SSH Port Forwarding:

SSH port forwarding allows us to securely tunnel traffic from a local port on our machine to a remote server over an SSH connection. This feature leverages SSH's encryption and authentication capabilities to create a secure channel for communication.

## Prerequisites:

-   Access to a local server behind NAT.
-   A public VPS with SSH access.

## Configuring SSH Port Forwarding:

First, we need to configure SSH port forwarding on the VPS to forward traffic from specific ports to our local server.

### Edit SSH Server Configuration:

Connect to your VPS via SSH and edit the SSH server configuration file (`/etc/ssh/sshd_config`).

```bash
sudo nano /etc/ssh/sshd_config
```

Find the `GatewayPorts` option and set it to yes to allow remote hosts to connect to forwarded ports.

```
GatewayPorts yes
```

Save the changes and restart the SSH service.

```bash
sudo systemctl restart sshd
```

### Configure Port Forwarding:

Next, establish port forwarding by executing the following command:

```bash
ssh -N -R <VPS_PUBLIC_IP>:<EXTERNAL_PORT>:<LOCAL_SERVER_IP>:<INTERNAL_PORT> user@<VPS_PUBLIC_IP>
```

Replace `<VPS_PUBLIC_IP>`, `<EXTERNAL_PORT>`, `<LOCAL_SERVER_IP>`, `<INTERNAL_PORT>`, and `user` with your actual values. Here's what each parameter represents:

-   `<VPS_PUBLIC_IP>`: Public IP of your VPS.
-   `<EXTERNAL_PORT>`: Port on the VPS that will be exposed to the internet.
-   `<LOCAL_SERVER_IP>`: Internal IP of your local server.
-   `<INTERNAL_PORT>`: Port on your local server that you want to expose.

## Creating a Systemd Service for Auto-Restart:

To ensure the port forwarding service automatically restarts every minute after a failure, we can create a systemd service.

### Create a Service File:

```bash
sudo nano /etc/systemd/system/ssh-port-forward.service
```

Add the following content to the file:

```ini
[Unit]
Description=SSH Port Forward Service
After=network.target

[Service]
User=<YOUR_USERNAME>
Group=<YOUR_GROUPNAME>
ExecStart=/usr/bin/ssh -N -R <VPS_PUBLIC_IP>:<EXTERNAL_PORT>:<LOCAL_SERVER_IP>:<INTERNAL_PORT> user@<VPS_PUBLIC_IP>
Restart=on-failure
RestartSec=1m

[Install]
WantedBy=multi-user.target
```

Replace placeholders `<YOUR_USERNAME>`, `<YOUR_GROUPNAME>`, `<VPS_PUBLIC_IP>`, `<EXTERNAL_PORT>`, `<LOCAL_SERVER_IP>`, `<INTERNAL_PORT>`, and `user` with your actual values.

### Enable and Start the Service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable ssh-port-forward
sudo systemctl start ssh-port-forward
```

## Conclusion:

Configuring SSH port forwarding allows us to securely expose local server ports to the internet via a public VPS. By setting up a systemd service with auto-restart functionality, we ensure continuous availability of the port forwarding service, enhancing reliability and convenience.

With these steps, you can securely expose services like RDP and SSH from your local server to the internet, enabling remote access while maintaining a high level of security. Always remember to implement best practices for securing your SSH configuration and regularly monitor system logs for any suspicious activities.
