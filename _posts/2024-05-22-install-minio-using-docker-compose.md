---
title: "Setting Up MinIO with Docker Compose"
date: 2024-05-22 00:01:00 +0700
categories: ["DevOps", "Containers"]
tags: ["Devops", "Docker", "MinIO", "Docker Compose"]
pin: false
---

### Setting Up MinIO with Docker Compose

MinIO is a high-performance, S3 compatible object storage system that you can self-host. It's a great tool for developers looking to implement their own storage solution. While running MinIO with a single `docker run` command is straightforward, using Docker Compose simplifies managing multiple services and their configurations. In this article, I'll show you how to set up MinIO using Docker Compose.

#### Prerequisites

Before we get started, ensure you have the following installed on your system:
- Docker
- Docker Compose

#### Step-by-Step Guide

##### Step 1: Prepare the Directory

First, you need to create a directory to store MinIO data. Open your terminal and execute the following command:

```bash
mkdir -p ~/minio/data
```

This command creates a directory called `minio/data` in your home directory.

##### Step 2: Create the Docker Compose File

Next, create a `docker-compose.yml` file. This file will define the MinIO service and its configuration. Navigate to the directory where you want to store this file and create the file using your preferred text editor:

```bash
nano docker-compose.yml
```

Paste the following content into the `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  minio:
        container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: ROOTNAME
      MINIO_ROOT_PASSWORD: CHANGEME123
    volumes:
      - ~/minio/data:/data
    command: server /data --console-address ":9001"
```

Here’s a breakdown of what each section does:

- **version**: Specifies the version of Docker Compose syntax.
- **services**: Defines the services to be run.
- **minio**: The name of our service.
  - **image**: The MinIO image from the container registry.
  - **container_name**: Name of the Docker container.
  - **ports**: Maps the container ports to your local machine.
  - **environment**: Sets the MinIO root user and password.
  - **volumes**: Maps a directory on your local machine to the container for persistent storage.
  - **command**: The command to start the MinIO server with the console accessible at port 9001.

##### Step 3: Start the MinIO Service

With your `docker-compose.yml` file in place, navigate to the directory containing the file and run the following command to start the MinIO service:

```bash
docker-compose up -d
```

The `-d` flag runs the containers in detached mode, meaning they will run in the background.

##### Step 4: Access the MinIO Console

Once the service is running, you can access the MinIO console by navigating to `http://localhost:9001` in your web browser. Use the credentials specified in the `docker-compose.yml` file (`ROOTNAME` and `CHANGEME123`) to log in.

#### Conclusion

Setting up MinIO with Docker Compose makes it easier to manage and configure your services, especially when working on larger projects or with multiple containers. By following this guide, you now have a working MinIO instance running on your local machine, ready for you to start storing and managing your data.

If you have any questions or run into issues, feel free to leave a comment below. Happy coding!
