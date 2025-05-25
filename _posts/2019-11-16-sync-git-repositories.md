---
title: "Effortlessly Migrate Git Repositories with Multiple Branches"
date: 2019-11-16 19:10:00 +0700
categories: ["DevOps", "Git"]
tags: ["DevOps", "Git", "Tools", "Scripts", "Repository Migration"]
pin: false
---

# Effortlessly Migrate Git Repositories with Multiple Branches

Migrating a Git repository to a new server can be daunting, especially when dealing with multiple branches. This guide simplifies the process with a step-by-step approach and a handy script to automate the migration.

## Why Automate Git Repository Migration?

Manually migrating branches one by one is time-consuming and error-prone. By automating the process, you can:

- Save time and effort.
- Ensure all branches are migrated accurately.
- Minimize downtime during the migration.

## Prerequisites

Before you begin, ensure you have:

- Access to both the source and destination Git servers.
- A local clone of the repository you want to migrate.

## Step-by-Step Guide

### 1. Clone the Original Repository

Start by cloning the repository from the source server:

```bash
git clone git://path-to-your-repo
cd your-repo
git fetch --all
```

### 2. Add the Destination Remote

Add the URL of the new repository as a remote:

```bash
git remote add dest git://path-to-your-remote-dest
```

### 3. List and Push Branches

Use the following script to list all branches from the source remote (`origin`) and push them to the destination remote (`dest`):

```bash
#!/bin/bash
# sync.sh: Migrate all branches from origin to dest

if [ "$#" -ne 3 ]; then
  echo "Usage: $0 path-to-directory origin dest"
  exit 1
fi

cd "$1" || exit
for branch in $(git branch -r | grep "$2/" | sed "s|$2/||"); do
  git push "$3" "$branch:$branch"
done
```

Save this script as `sync.sh` and make it executable:

```bash
chmod +x sync.sh
```

Run the script:

```bash
./sync.sh path-to-directory origin dest
```

### 4. Verify the Migration

After the script completes, verify that all branches have been migrated:

```bash
git branch -r
```

## Conclusion

By following this guide, you can migrate Git repositories with multiple branches quickly and efficiently. Automating the process ensures accuracy and saves valuable time, making it an essential skill for DevOps professionals.
