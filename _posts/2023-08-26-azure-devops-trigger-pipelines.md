---
title: Automate Triggering All Pipelines in Azure DevOps
date: 2023-08-26 10:10:00 +0700
categories: [DevOps, Azure DevOps]
tags: [DevOps, Azure DevOps, Automation, Scripts]
pin: false
---

Azure DevOps is a powerful platform for managing your software development lifecycle. Automating pipeline triggers can save time and ensure consistency across builds, tests, and deployments. In this guide, we'll explore how to trigger all pipelines in an Azure DevOps project using the REST API and scripting.

## Prerequisites

Before proceeding, ensure you have:

- **Azure DevOps Account**: An active account with permissions to manage pipelines.
- **Personal Access Token (PAT)**: A PAT with "Build (Read and Execute)" and "Release (Read and Execute)" permissions.
- **Pipeline Configuration**: Pipelines set up in your Azure DevOps project.

## Why Automate Pipeline Triggers?

Automating pipeline triggers is useful for scenarios like:

- Scheduled releases.
- Ensuring consistent builds across multiple branches.
- Running all pipelines after major updates.

## Step-by-Step Guide

### Step 1: Generate a Personal Access Token (PAT)

1. Log in to your Azure DevOps account.
2. Go to **Profile > Security > Personal Access Tokens**.
3. Click **New Token**, configure permissions, and generate the PAT.
4. Save the PAT securely.

### Step 2: Use REST API to Trigger Pipelines

The Azure DevOps REST API provides endpoints to list and trigger pipelines. Here's how to automate this process:

#### Fetch Pipeline IDs

Use the following endpoint to list all pipelines:

```bash
GET https://dev.azure.com/{organization}/{project}/_apis/pipelines?api-version=7.0
```

#### Trigger Pipelines

Use this endpoint to trigger a specific pipeline:

```bash
POST https://dev.azure.com/{organization}/{project}/_apis/pipelines/{pipelineId}/runs?api-version=7.0
```

### Step 3: Automate with a Script

Below is a Bash script to fetch pipeline IDs and trigger them sequentially:

```bash
#!/bin/bash

# Azure DevOps Configuration
pat="YOUR_PERSONAL_ACCESS_TOKEN"
organization="YOUR_ORGANIZATION"
project="YOUR_PROJECT"
apiVersion="7.0"

# Fetch all pipeline IDs
pipelines=$(curl -s -H "Authorization: Basic $(echo -n :$pat | base64)" \
    "https://dev.azure.com/$organization/$project/_apis/pipelines?api-version=$apiVersion")

# Extract pipeline IDs
pipeline_ids=$(echo "$pipelines" | jq -r '.value[].id')

# Trigger each pipeline
for pipeline_id in $pipeline_ids; do
    echo "Triggering pipeline ID: $pipeline_id"
    curl -s -H "Authorization: Basic $(echo -n :$pat | base64)" \
         -X POST \
         "https://dev.azure.com/$organization/$project/_apis/pipelines/$pipeline_id/runs?api-version=$apiVersion" \
         --data "{}"
done
```

**Script Notes**:
- Replace `YOUR_PERSONAL_ACCESS_TOKEN`, `YOUR_ORGANIZATION`, and `YOUR_PROJECT` with actual values.
- The script uses `jq` to parse JSON responses. Install it via `brew install jq` if not already installed.

## Conclusion

By automating pipeline triggers with the Azure DevOps REST API, you can streamline your CI/CD processes and ensure consistent builds across your project. This approach is especially useful for large projects with multiple pipelines.
