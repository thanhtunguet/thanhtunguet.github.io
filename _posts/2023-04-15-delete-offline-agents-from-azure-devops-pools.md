---
title: Delete offline agents from a pool on Azure Devops Server
date: 2023-04-15 10:10:00 +0700
categories: [Devops, Azure Devops]
tags: [Devops, Azure Devops, Scripts]
pin: false
---

# Prerequisites

Before getting started, make sure you have the following:

- An Azure DevOps organization URL
- A Personal Access Token (PAT) with permission to manage agents
- A working Bash environment

## Step 1: Set the Azure DevOps organization URL and PAT token

Open your preferred text editor and create a new Bash script. At the top of the script, set the `ORG_URL` variable to your Azure DevOps organization URL and the `PAT_TOKEN` variable to your PAT.

```bash
#!/usr/bin/env bash

# Set your Azure DevOps organization URL and PAT token
ORG_URL="https://devops-server.example.com/DefaultCollection"
PAT_TOKEN="<PAT_TOKEN>"
```

# Step 2: Encode the PAT token

To use the PAT token for authentication, we need to encode it as base64. We can do this using the base64 command.

```bash
# Encode the PAT token as base64 for use in the Authorization header
PAT_TOKEN=$(printf "%s"":$PAT_TOKEN" | base64)
```

# Step 3: Set the agent pool ID and API version

The script takes the agent pool ID as an argument when it is run. We can set the pool ID and API version as variables.

```bash
# Set the agent pool ID and API version
POOL_ID="$1"
API_VERSION="6.0"
```

## Step 4: Retrieve a list of all agents in the pool

We can use the curl command to retrieve a list of all agents in the specified pool. We pass the PAT token and API version as headers and store the result in the AGENTS variable.

```bash
# Get a list of all agents in the pool
AGENTS=$(curl -s -H "Authorization: Basic $PAT_TOKEN" "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents?api-version=${API_VERSION}")
```

## Step 5: Loop through each agent and delete the offline agents

We can use the jq command to loop through each agent in the retrieved list and retrieve its status. If an agent's status is "offline," we can send a `DELETE` request to the API to delete the agent from the pool.

```bash
# Loop through each agent and delete the offline agents
for AGENT in $(echo $AGENTS | jq -r '.value[].id')
do
  STATUS=$(echo $AGENTS | jq -r --arg AGENTID "$AGENT" '.value[] | select(.id == ($AGENTID | tonumber)) | .status')

  if [ "$STATUS" == "offline" ]
  then
    echo "Deleting agent with ID $AGENT"
    curl -s -X DELETE -H "Authorization: Basic $PAT_TOKEN" "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents/${AGENT}?api-version=${API_VERSION}"
  fi
done
```

## Step 6: Run the script

Save the script with a descriptive name, such as `azure-devops-delete-offline-agents.sh`. Make the script executable using the chmod command.

```bash
chmod +x azure-devops-delete-offline-agents.sh
```

To run the script, pass the agent pool ID as an argument.

```bash
./azure-devops-delete-offline-agents.sh <pool_id>
```

## Full Script

Here's the complete script with all the steps combined.

```bash
#!/usr/bin/env bash

# Set your Azure DevOps organization URL and PAT token
ORG_URL="https://devops-server.example.com/DefaultCollection"
PAT_TOKEN="<PAT_TOKEN>"
PAT_TOKEN=$(printf "%s"":$PAT_TOKEN" | base64)

# Set the agent pool ID and API version
POOL_ID="$1"
API_VERSION="6.0"

# Get a list of all agents in the pool
AGENTS=$(curl -s -H "Authorization: Basic $PAT_TOKEN" "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents?api-version=${API_VERSION}")

# Loop through each agent and delete the offline agents
for AGENT in $(echo $AGENTS | jq -r '.value[].id')
do
  STATUS=$(echo $AGENTS | jq -r --arg AGENTID "$AGENT" '.value[] | select(.id == ($AGENTID | tonumber)) | .status')

  if [ "$STATUS" == "offline" ]
  then
    echo "Deleting agent with ID $AGENT"
    curl -s -X DELETE -H "Authorization: Basic $PAT_TOKEN" "${ORG_URL}/_apis/distributedtask/pools/${POOL_ID}/agents/${AGENT}?api-version=${API_VERSION}"
  fi
done
```
