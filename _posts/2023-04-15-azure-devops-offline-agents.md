---
title: Automating the Cleanup of Offline Agents in Azure DevOps
date: 2023-04-15 10:10:00 +0700
categories: [DevOps, Azure DevOps]
tags: [DevOps, Azure DevOps, Automation, Scripts]
pin: false
---

Managing agent pools in Azure DevOps is crucial for maintaining an efficient CI/CD pipeline. Offline agents can clutter your pool and cause unnecessary delays. This guide provides a script to automate the cleanup of offline agents.

## Prerequisites

Ensure you have:

- **Azure DevOps Organization URL**: The URL of your Azure DevOps instance.
- **Personal Access Token (PAT)**: A PAT with permissions to manage agents.
- **Bash Environment**: A working Bash shell with `curl` and `jq` installed.

## Step-by-Step Guide

### Step 1: Configure the Script

Create a Bash script and configure the following variables:

```bash
#!/usr/bin/env bash

# Azure DevOps Configuration
ORG_URL="https://devops-server.example.com/DefaultCollection"
PAT_TOKEN="<PAT_TOKEN>"
POOL_ID="$1"  # Pass the agent pool ID as an argument
API_VERSION="6.0"

# Encode the PAT token
PAT_TOKEN=$(printf "%s"":$PAT_TOKEN" | base64)
```

### Step 2: Retrieve Agents in the Pool

Use the following command to fetch all agents in the specified pool:

```bash
AGENTS=$(curl -s -H "Authorization: Basic $PAT_TOKEN" \
    "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents?api-version=${API_VERSION}")
```

### Step 3: Delete Offline Agents

Loop through the agents and delete those with a status of "offline":

```bash
for AGENT in $(echo $AGENTS | jq -r '.value[].id'); do
  STATUS=$(echo $AGENTS | jq -r --arg AGENTID "$AGENT" \
    '.value[] | select(.id == ($AGENTID | tonumber)) | .status')

  if [ "$STATUS" == "offline" ]; then
    echo "Deleting agent with ID $AGENT"
    curl -s -X DELETE -H "Authorization: Basic $PAT_TOKEN" \
        "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents/${AGENT}?api-version=${API_VERSION}"
  fi
done
```

### Step 4: Run the Script

Save the script as `delete-offline-agents.sh`, make it executable, and run it:

```bash
chmod +x delete-offline-agents.sh
./delete-offline-agents.sh <pool_id>
```

## Full Script

```bash
#!/usr/bin/env bash

# Azure DevOps Configuration
ORG_URL="https://devops-server.example.com/DefaultCollection"
PAT_TOKEN="<PAT_TOKEN>"
PAT_TOKEN=$(printf "%s"":$PAT_TOKEN" | base64)
POOL_ID="$1"
API_VERSION="6.0"

# Fetch agents
AGENTS=$(curl -s -H "Authorization: Basic $PAT_TOKEN" \
    "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents?api-version=${API_VERSION}")

# Delete offline agents
for AGENT in $(echo $AGENTS | jq -r '.value[].id'); do
  STATUS=$(echo $AGENTS | jq -r --arg AGENTID "$AGENT" \
    '.value[] | select(.id == ($AGENTID | tonumber)) | .status')

  if [ "$STATUS" == "offline" ]; then
    echo "Deleting agent with ID $AGENT"
    curl -s -X DELETE -H "Authorization: Basic $PAT_TOKEN" \
        "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents/${AGENT}?api-version=${API_VERSION}"
  fi
done
```

## Conclusion

This script simplifies the process of managing offline agents in Azure DevOps, ensuring your agent pools remain clean and efficient. Automating this task can save time and reduce manual effort.
