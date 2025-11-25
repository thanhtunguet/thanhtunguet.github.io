---
title: "Exporting and Cleaning Kubernetes Resources: A Practical Guide"
date: 2025-09-25 10:32:00 +0700
categories: ["DevOps", "Kubernetes"]
tags: ["DevOps", "Kubernetes"]
pin: false
---

If you've ever tried to migrate Kubernetes resources between clusters or create reusable configuration templates from running workloads, you've likely encountered a frustrating problem: exported YAML files are cluttered with cluster-specific metadata that prevents clean re-deployment. When you run `kubectl get deployment my-app -o yaml`, you get a mountain of runtime information that's only relevant to the source cluster—resourceVersions, UIDs, timestamps, and more.

In this article, I'll walk you through a practical two-script solution that automates the export and cleaning of Kubernetes resources. Whether you're backing up configurations, migrating between environments, or setting up GitOps workflows, these scripts will save you hours of manual YAML editing.

## The Problem: Cluster-Specific Metadata

When you export Kubernetes resources using `kubectl get -o yaml`, you get the complete object representation including all the metadata Kubernetes has added during runtime. While this is useful for inspection and debugging, it creates problems when you want to apply these manifests to a different cluster.

Here are the problematic fields that clutter your exported YAML:

**Runtime Metadata:**
- `resourceVersion` - Kubernetes' internal versioning for optimistic concurrency control
- `uid` - Unique identifier assigned by the cluster
- `generation` - Tracks specification changes
- `creationTimestamp` - When the resource was originally created

**Tracking Annotations:**
- `kubectl.kubernetes.io/last-applied-configuration` - Stores the entire previous configuration for three-way merge
- `cattle.io/timestamp` - Rancher-specific tracking information
- `kubectl.kubernetes.io/restartedAt` - Restart tracking annotations

**Service-Specific Fields:**
- `clusterIP` and `clusterIPs` - Assigned by the cluster's service network

**Status Objects:**
- The entire `status` section containing runtime state information

These fields cause several issues:
1. They make manifests specific to the source cluster
2. They create unnecessary diffs in version control
3. They can cause validation errors when applying to new clusters
4. They bloat file sizes with redundant information

The solution? A two-step process: export resources systematically, then clean them programmatically.

## Solution Part 1: The Export Script

Let's start with a bash script that exports resources from a namespace in an organized way:

```bash
#!/usr/bin/env bash
set -euo pipefail

NAMESPACE="$1"
OUTPUT_DIR="$2"

if [[ -z "$NAMESPACE" || -z "$OUTPUT_DIR" ]]; then
  echo "Usage: $0 <namespace> <output-folder>"
  exit 1
fi

mkdir -p "$OUTPUT_DIR"

# List of resource types to export
RESOURCES=(
  configmaps
  secrets
  ingresses
  deployments
  services
)

for resource in "${RESOURCES[@]}"; do
  echo "Exporting $resource..."

  # Get all resource names in the namespace
  kubectl get "$resource" -n "$NAMESPACE" -o jsonpath="{.items[*].metadata.name}" | tr ' ' '\n' | while read -r name; do
    if [[ -n "$name" ]]; then
      OUTPUT_FILE="${OUTPUT_DIR}/${resource}-${name}.yaml"
      echo "  -> $OUTPUT_FILE"
      kubectl get "$resource" "$name" -n "$NAMESPACE" -o yaml > "$OUTPUT_FILE"
    fi
  done
done

echo "Done! YAML files saved in $OUTPUT_DIR"
```

### How It Works

The script takes two arguments: the namespace to export from and the output directory for the YAML files. It uses `set -euo pipefail` for robust error handling—the script will exit immediately if any command fails.

The heart of the script is the `RESOURCES` array, which defines which resource types to export. I've included the most common resources (ConfigMaps, Secrets, Ingresses, Deployments, and Services), but you can easily extend this list to include StatefulSets, DaemonSets, PersistentVolumeClaims, or any other resource type.

For each resource type, the script:
1. Lists all resources of that type in the namespace
2. Iterates through each resource name
3. Exports each resource to its own YAML file with a clear naming convention: `{resource-type}-{resource-name}.yaml`

### Benefits of Individual File Exports

Exporting each resource to a separate file (rather than one massive YAML file) provides several advantages:

- **Version control friendly** - Each resource change creates a focused diff
- **Selective deployment** - Apply only the resources you need
- **Easier review** - Quickly locate and examine specific resources
- **Merge conflict reduction** - Team members can work on different resources without conflicts

### Usage Example

```bash
# Make the script executable
chmod +x export-k8s-resources.sh

# Export all resources from the 'production' namespace
./export-k8s-resources.sh production ./output

# The output directory now contains files like:
# - configmaps-app-config.yaml
# - secrets-db-credentials.yaml
# - deployments-web-app.yaml
# - services-web-app.yaml
# - ingresses-main-ingress.yaml
```

## Solution Part 2: The Cleanup Script

Now that we have exported YAML files, we need to clean them. Here's where a Node.js script comes in handy for its excellent YAML parsing capabilities:

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');
const yaml = require('js-yaml');

// Directory containing YAML files
const OUTPUT_DIR = path.join(__dirname, 'output');

// Fields to remove from metadata
const METADATA_FIELDS_TO_REMOVE = [
  'creationTimestamp',
  'resourceVersion',
  'uid',
  'generation'
];

// Annotations to remove
const ANNOTATIONS_TO_REMOVE = [
  'kubectl.kubernetes.io/last-applied-configuration',
  'cattle.io/timestamp',
  'kubectl.kubernetes.io/restartedAt'
];

function cleanK8sYaml(doc) {
  // Remove status object at root level
  if (doc.status) {
    delete doc.status;
  }

  // Clean metadata
  if (doc.metadata) {
    METADATA_FIELDS_TO_REMOVE.forEach(field => {
      if (doc.metadata[field]) {
        delete doc.metadata[field];
      }
    });

    // Clean annotations
    if (doc.metadata.annotations) {
      ANNOTATIONS_TO_REMOVE.forEach(annotation => {
        if (doc.metadata.annotations[annotation]) {
          delete doc.metadata.annotations[annotation];
        }
      });

      // Remove annotations object if it's empty
      if (Object.keys(doc.metadata.annotations).length === 0) {
        delete doc.metadata.annotations;
      }
    }

    // Clean template metadata (for Deployments, StatefulSets, etc.)
    if (doc.spec && doc.spec.template && doc.spec.template.metadata) {
      if (doc.spec.template.metadata.creationTimestamp !== undefined) {
        delete doc.spec.template.metadata.creationTimestamp;
      }

      if (doc.spec.template.metadata.annotations) {
        ANNOTATIONS_TO_REMOVE.forEach(annotation => {
          if (doc.spec.template.metadata.annotations[annotation]) {
            delete doc.spec.template.metadata.annotations[annotation];
          }
        });

        if (Object.keys(doc.spec.template.metadata.annotations).length === 0) {
          delete doc.spec.template.metadata.annotations;
        }
      }
    }
  }

  // Clean Service-specific fields
  if (doc.kind === 'Service' && doc.spec) {
    if (doc.spec.clusterIP !== undefined) {
      delete doc.spec.clusterIP;
    }
    if (doc.spec.clusterIPs !== undefined) {
      delete doc.spec.clusterIPs;
    }
  }

  return doc;
}

function processYamlFile(filePath) {
  try {
    const content = fs.readFileSync(filePath, 'utf8');
    const docs = yaml.loadAll(content);
    const cleanedDocs = docs.map(doc => cleanK8sYaml(doc));
    
    const cleanedContent = cleanedDocs
      .map(doc => yaml.dump(doc, {
        lineWidth: -1,
        noRefs: true,
        sortKeys: false
      }))
      .join('---\n');

    fs.writeFileSync(filePath, cleanedContent, 'utf8');
    console.log(`✓ Cleaned: ${path.basename(filePath)}`);
    return true;
  } catch (error) {
    console.error(`✗ Error processing ${path.basename(filePath)}:`, error.message);
    return false;
  }
}

function main() {
  if (!fs.existsSync(OUTPUT_DIR)) {
    console.error(`Error: Output directory not found: ${OUTPUT_DIR}`);
    process.exit(1);
  }

  const files = fs.readdirSync(OUTPUT_DIR)
    .filter(file => file.endsWith('.yaml') || file.endsWith('.yml'))
    .map(file => path.join(OUTPUT_DIR, file));

  if (files.length === 0) {
    console.log('No YAML files found in output directory');
    return;
  }

  console.log(`Found ${files.length} YAML file(s) to clean...\n`);

  let successCount = 0;
  let errorCount = 0;

  files.forEach(file => {
    if (processYamlFile(file)) {
      successCount++;
    } else {
      errorCount++;
    }
  });

  console.log(`\nDone! Successfully cleaned ${successCount} file(s)`);
  if (errorCount > 0) {
    console.log(`Failed to clean ${errorCount} file(s)`);
  }
}

main();
```

### What Gets Removed and Why

The cleanup script systematically removes several categories of fields:

**Metadata Fields** - These are cluster-assigned identifiers and timestamps that have no meaning in a different cluster. The new cluster will generate its own values when you apply the manifest.

**Tracking Annotations** - The `last-applied-configuration` annotation can be enormous (often containing the entire resource definition) and serves no purpose in a fresh deployment. Rancher-specific annotations like `cattle.io/timestamp` are only relevant to the source cluster's management tooling.

**Status Objects** - The `status` field contains runtime state information (pod counts, conditions, observed generation, etc.). This is always managed by Kubernetes controllers and should never be in declarative manifests.

**Service Network Fields** - For Services, the `clusterIP` and `clusterIPs` fields are assigned from the cluster's service CIDR range. Including these can cause conflicts or validation errors in the target cluster.

**Template Metadata** - For workload resources like Deployments and StatefulSets, the script also cleans the `spec.template.metadata` section, removing timestamps and annotations that might be present in pod templates.

### How the Cleaning Logic Works

The script uses the `js-yaml` library to parse YAML files safely. One powerful feature is support for multi-document YAML files (documents separated by `---`). Using `yaml.loadAll()` ensures that even if a file contains multiple resources, each one gets cleaned properly.

The `cleanK8sYaml()` function works methodically through each document:

1. **Remove status** - Simple deletion at the root level
2. **Clean metadata** - Iterate through unwanted fields and delete them
3. **Clean annotations** - Remove specific annotations, then delete the entire annotations object if it becomes empty
4. **Handle templates** - For resources that have pod templates (Deployments, StatefulSets, Jobs), clean the template metadata too
5. **Resource-specific cleaning** - Special handling for Services to remove cluster IP assignments

After cleaning, the script converts the documents back to YAML using specific formatting options:
- `lineWidth: -1` prevents line wrapping
- `noRefs: true` avoids YAML anchors and aliases
- `sortKeys: false` preserves the original key order

### Script Execution

```bash
# Install dependencies first
npm install js-yaml

# Make the script executable
chmod +x clean-k8s-yaml.js

# Run the cleanup
./clean-k8s-yaml.js

# Output:
# Found 12 YAML file(s) to clean...
# 
# ✓ Cleaned: configmaps-app-config.yaml
# ✓ Cleaned: secrets-db-credentials.yaml
# ✓ Cleaned: deployments-web-app.yaml
# ...
# 
# Done! Successfully cleaned 12 file(s)
```

## Putting It All Together: Complete Workflow

Here's the complete workflow from export to cleaned, deployable manifests:

### Prerequisites

```bash
# Ensure you have kubectl configured
kubectl cluster-info

# Verify Node.js is installed (v12 or higher)
node --version

# Install the YAML parsing library
npm install js-yaml
```

### Step-by-Step Execution

```bash
# Step 1: Create a workspace
mkdir k8s-backup
cd k8s-backup

# Step 2: Save both scripts
# (Save the bash script as export-k8s-resources.sh)
# (Save the Node.js script as clean-k8s-yaml.js)

# Step 3: Make scripts executable
chmod +x export-k8s-resources.sh
chmod +x clean-k8s-yaml.js

# Step 4: Export resources
./export-k8s-resources.sh my-namespace ./output

# Step 5: Clean the exported files
./clean-k8s-yaml.js

# Step 6: Review the cleaned files
ls -lh output/
```

### What the Final Files Look Like

Before cleaning, a Deployment export might look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment"...} # Huge blob
  creationTimestamp: "2024-01-15T10:30:00Z"
  generation: 3
  name: web-app
  namespace: production
  resourceVersion: "12345678"
  uid: a1b2c3d4-e5f6-7890-abcd-ef1234567890
spec:
  replicas: 3
  # ... spec continues
status:
  availableReplicas: 3
  conditions: # ... lots of status info
```

After cleaning, it becomes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
  name: web-app
  namespace: production
spec:
  replicas: 3
  # ... spec continues (no status section)
```

Much cleaner! Now you can apply this to any cluster without issues:

```bash
# Apply to a new cluster
kubectl apply -f output/
```

## Customization and Extension

The scripts are designed to be easily customizable for your specific needs.

### Adding More Resource Types

Simply extend the `RESOURCES` array in the bash script:

```bash
RESOURCES=(
  configmaps
  secrets
  ingresses
  deployments
  services
  statefulsets
  daemonsets
  persistentvolumeclaims
  serviceaccounts
  roles
  rolebindings
)
```

### Customizing Field Removal

Need to keep certain annotations or remove additional fields? Just modify the arrays in the Node.js script:

```javascript
// Keep specific annotations
const ANNOTATIONS_TO_REMOVE = [
  'kubectl.kubernetes.io/last-applied-configuration',
  'cattle.io/timestamp',
  // Add your own here
];

// Remove additional metadata fields
const METADATA_FIELDS_TO_REMOVE = [
  'creationTimestamp',
  'resourceVersion',
  'uid',
  'generation',
  'managedFields',  // Add this if you want to remove managed fields
];
```

### Handling Namespace-Specific Configurations

If you want to make the exported resources more portable across namespaces, you could add logic to:
- Remove the namespace field from metadata
- Replace namespace references in configuration with placeholders
- Generate a separate namespace manifest

### Version Control Integration

These cleaned manifests are perfect for Git:

```bash
# Initialize a Git repository for your configs
git init
git add output/*.yaml
git commit -m "Initial export from production cluster"

# Track changes over time
./export-k8s-resources.sh production ./output
./clean-k8s-yaml.js
git diff  # See what changed
git commit -am "Updated configurations"
```

## Best Practices and Considerations

### When to Use This Approach

This export-and-clean approach works best for:

- **One-time migrations** - Moving workloads between clusters
- **Configuration backups** - Regular snapshots of your cluster state
- **Reverse engineering** - Creating manifests from existing deployments
- **Documentation** - Maintaining records of deployed configurations

However, for new projects, consider using higher-level tools:

- **Helm** - Package manager with templating and versioning
- **Kustomize** - Native Kubernetes configuration management
- **GitOps tools** - ArgoCD or Flux for automated deployments

### Security Considerations

Be extremely careful with Secrets:

```bash
# Secrets are exported as base64-encoded data, NOT encrypted!
# Never commit secrets to public repositories

# Option 1: Exclude secrets from version control
echo "output/secrets-*.yaml" >> .gitignore

# Option 2: Use sealed secrets or external secret management
# - Sealed Secrets (Bitnami)
# - External Secrets Operator
# - HashiCorp Vault integration
```

### Testing Before Production

Always test cleaned manifests in a non-production environment:

```bash
# Apply to a test namespace first
kubectl create namespace test-migration
kubectl apply -f output/ -n test-migration

# Verify everything works
kubectl get all -n test-migration

# Check logs for errors
kubectl logs -n test-migration deployment/web-app

# Clean up test
kubectl delete namespace test-migration
```

### Backup Strategies

Consider automating regular exports:

```bash
#!/usr/bin/env bash
# backup-cron.sh

DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_DIR="./backups/$DATE"

./export-k8s-resources.sh production "$BACKUP_DIR"
cd "$BACKUP_DIR" && ../../clean-k8s-yaml.js

# Keep only last 30 days of backups
find ./backups -type d -mtime +30 -exec rm -rf {} \;
```

## Conclusion

Exporting and cleaning Kubernetes resources doesn't have to be a manual, error-prone process. With these two simple scripts, you can automate the entire workflow and get clean, reusable manifests that are ready to apply to any cluster.

The bash export script gives you organized, individual YAML files for each resource, making version control and selective deployment straightforward. The Node.js cleanup script removes all the cluster-specific cruft that Kubernetes adds during runtime, leaving you with pristine declarative configurations.

This approach shines in several scenarios:

- **Disaster recovery** - Quick restoration of configurations to a new cluster
- **Cluster migration** - Moving workloads between cloud providers or on-premises infrastructure
- **GitOps workflows** - Maintaining infrastructure-as-code repositories
- **Development environments** - Replicating production configurations in staging or local clusters

The scripts are intentionally simple and transparent, making them easy to understand, customize, and extend for your specific needs. Whether you're managing a handful of microservices or hundreds of resources across multiple namespaces, this workflow will save you countless hours of manual YAML editing.

Feel free to adapt these scripts to your environment—add resource types, customize the cleaning logic, or integrate them into your CI/CD pipelines. The goal is to make Kubernetes configuration management less painful and more reproducible.
