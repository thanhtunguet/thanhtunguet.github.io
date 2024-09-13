---
title: "How to sync container images to your private registry"
date: 2024-01-21 07:45:00 +0700
categories: [Devops, Docker]
tags: [Devops, Linux, Docker, Private Registry]
pin: false
image: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRDJHmIjtpUQRNbX82alVhC7sRSs3pZw8Pklg&s
---

Introduction
------------

In the fast-paced world of software development, managing Docker container images efficiently is crucial for smooth operations and deployment. A common challenge faced by DevOps teams is syncing container images between different registries, such as moving from a public registry to a private one for security, compliance, or performance reasons. Manually pulling, tagging, and pushing each image to a new registry can be time-consuming and error-prone. To address this challenge, I'll introduce a Bash script that automates the process, making it both efficient and reliable.

The Problem: Manual Image Transfer
----------------------------------

Traditionally, transferring Docker images from one registry to another involves several manual steps:

1.  **Pulling the Image**: You need to pull each image from the original registry to your local machine.
2.  **Tagging the Image**: After pulling an image, you must tag it with the new registry's address.
3.  **Pushing the Image**: Finally, you push the tagged image to the new registry.

Repeating these steps for numerous images is not only tedious but also prone to human error.

The Solution: An Automated Bash Script
--------------------------------------

To streamline this process, I've developed a Bash script that automates these tasks. This script reads a list of Docker images from a file and performs the pull, tag, and push operations for each image. It's designed to be simple to use, requiring only the image list file and the new registry domain as inputs.

## Script Overview

Here's an overview of how the script works:

1.  **Specify the Image List and New Registry**: You provide a file containing a list of Docker images (one per line) and the domain of the new registry as arguments to the script.

2.  **Automated Image Transfer**: The script reads the file and for each image, performs the following:

    -   Pulls the image from its current registry.
    -   Tags the image with the new registry's domain.
    -   Pushes the image to the new registry.

3.  **Optional Cleanup**: After transferring an image, the script can optionally remove the locally pulled image to save space.

## The Script

Here's the Bash script that accomplishes this task:

```bash
#!/bin/bash

# File containing the list of Docker images (one image per line)
IMAGE_LIST_FILE="$1"

# New registry domain
NEW_REGISTRY="$2"

# Read the image list file line by line
while IFS= read -r image || [[ -n "$image" ]]; do
    # Pull the original image
    docker pull "$image"

    # Extract the image name without the registry/domain
    image_name="${image#*/}"

    # Tag the image with the new registry domain
    new_image="$NEW_REGISTRY/$image_name"
    docker tag "$image" "$new_image"

    # Push the tagged image to the new registry
    docker push "$new_image"

    # Optional: Remove the locally pulled images
    # docker rmi "$image"
    # docker rmi "$new_image"
done < "$IMAGE_LIST_FILE"
```

## How to Use the Script

1.  **Prepare Your Image List File**: Create a file listing the Docker images you want to transfer, with one image per line.

2.  **Run the Script**: Execute the script with the image list file and the new registry domain as arguments.

    ```bash
    ./sync_images.sh image_list.txt myregistry.example.com
    ```

3. **Monitor the Process**: The script will output the status of each operation, allowing you to monitor the progress.

## Conclusion

This script significantly simplifies the process of syncing Docker images between registries. It's a practical solution for DevOps professionals looking to automate and streamline their container image management workflows.

