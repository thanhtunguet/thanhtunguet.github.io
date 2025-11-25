---
title: "Managing Storage Consumption on CI/CD Agent Machines: A Practical Guide"
date: 2024-08-14 15:30:00 +0700
categories: ["DevOps", "CI/CD"]
tags: ["DevOps", Linux]
pin: false
mermaid: true
---

## Managing Storage Consumption on CI/CD Agent Machines: A Practical Guide

Continuous Integration and Continuous Deployment (CI/CD) pipelines are the backbone of modern software development processes. They automate the testing, integration, and deployment of applications, making the process faster, more reliable, and more efficient. However, one of the challenges that often arises with CI/CD agent machines, particularly those running on Linux, is managing storage consumption.

### The Problem: Storage Bloat on CI/CD Agents

CI/CD agent machines are typically tasked with running multiple builds, tests, and deployments. Over time, these processes can consume a significant amount of disk space. Artifacts such as Docker images, containers, logs, and temporary files accumulate, leading to storage bloat. This can cause a range of issues, from slower performance due to disk I/O bottlenecks to complete failures of builds or deployments when the disk space is exhausted.

When disk space becomes scarce, it can lead to interruptions in your CI/CD pipeline, potentially delaying the delivery of your software. Therefore, it’s crucial to monitor and manage storage usage on these machines to ensure they continue to function optimally.

### The Solution: Automated Disk Space Management Script

To tackle this issue, I’ve created a simple yet effective Bash script that automatically monitors the disk usage of a specified volume and triggers a cleanup process when the usage exceeds a defined threshold. This script specifically targets Docker-related storage, which is often a major contributor to disk space consumption on CI/CD agents.

Here’s the script:

```bash
#!/bin/bash

# Set the threshold for storage usage
THRESHOLD=90

# Get the current usage percentage of the specified volume
usage=$(df -h /dev/mapper/vgubuntu-root | awk 'NR==2 {print $5}' | sed 's/%//')

currentDate=$(date +"%Y-%m-%d %H:%M:%S")

# Check if usage is greater than or equal to the threshold
if [ $usage -ge $THRESHOLD ]; then
    echo "$currentDate Storage usage is above $THRESHOLD% on /dev/mapper/vgubuntu-root. Running docker system prune to clean up."
    # Run docker system prune command
    docker system prune -a -f --volumes
else
    echo "$currentDate Storage usage on /dev/mapper/vgubuntu-root is $usage%, below $THRESHOLD%. No action needed."
fi
```

### How It Works

1. **Threshold Setting**: The script begins by setting a storage usage threshold (e.g., 90%). This threshold determines when the cleanup process should be triggered.

2. **Usage Monitoring**: The script then checks the current disk usage of the specified volume (`/dev/mapper/vgubuntu-root` in this case). This is done using the `df -h` command, which reports disk space usage in a human-readable format. The script extracts the usage percentage using `awk` and `sed`.

3. **Conditional Cleanup**: If the disk usage exceeds the defined threshold, the script logs the event and runs the `docker system prune -a -f --volumes` command. This command aggressively cleans up unused Docker objects, including images, containers, volumes, and networks, freeing up significant disk space.

4. **Logging**: The script logs each check with a timestamp, indicating whether a cleanup was performed or if no action was needed.

### Why This Script Matters

By automating the monitoring and cleanup process, this script helps maintain the health of your CI/CD agent machines. It ensures that disk space is managed proactively, reducing the risk of pipeline disruptions due to storage issues. 

Moreover, the script is simple and easy to customize. You can adjust the threshold, change the target volume, or modify the cleanup command to suit your specific environment.

### Conclusion

Managing storage consumption on CI/CD agent machines is a critical task that, if overlooked, can lead to significant issues in your development pipeline. This script provides a straightforward solution to keep your agent machines running smoothly by automatically cleaning up Docker-related storage when needed.

By implementing such automated solutions, you can focus more on building and deploying great software, knowing that your infrastructure is well-maintained and less prone to storage-related issues.
