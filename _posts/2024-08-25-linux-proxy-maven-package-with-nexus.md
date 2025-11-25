---
title: "Why Proxying Maven Packages with Nexus on Your Own Server is Essential"
date: 2024-08-25 15:30:00 +0700
categories: ["DevOps", "Infrastructure"]
tags: ["DevOps", Linux, "Android", "Mobile Development"]
pin: false
mermaid: true
---

In today's fast-paced development environment, managing dependencies efficiently and securely is crucial. Whether you're working on Java, Android, or any Maven-based projects, relying on external repositories like Maven Central or JCenter is a common practice. However, this approach comes with certain risks and inefficiencies. To mitigate these issues, a powerful solution is to use Sonatype Nexus to proxy all Maven packages on your own premise server.

This article introduces an approach to setting up a Nexus repository to proxy all your Maven dependencies locally, explains why this practice is necessary, provides a script to automate the process, and outlines how to configure your Gradle build to use these local repositories.

### Why Proxying Maven Repositories Locally is Necessary

#### 1. **Optimizing Load Times**

When your build tool (Gradle, Maven, etc.) fetches dependencies from remote repositories, network latency and external traffic can significantly slow down your build times. By proxying these repositories locally with Nexus, you can cache all necessary dependencies on your server. This local caching drastically reduces the time it takes to resolve dependencies, leading to faster, more efficient builds.

#### 2. **Ensuring Continuous Access**

Relying solely on public repositories can be risky. Packages may become outdated, unpublished, or inaccessible for various reasons:
- **Package Unpublished:** A package owner might remove their package from the public repository, leaving you without access.
- **Repository Shutdown:** Public repositories like JCenter have been known to shut down, disrupting the availability of essential packages.
- **Access Restrictions:** Free repositories may switch to paid access or impose stricter access controls, limiting your ability to download dependencies.

With a locally proxied repository, all dependencies are stored on your server. Even if the original repository becomes unavailable, your builds can continue to access the cached packages, ensuring uninterrupted development.

#### 3. **Enhancing Security**

Security is a significant concern when using public repositories. Dependencies downloaded directly from public sources may be compromised, containing malicious code or vulnerabilities. By proxying these repositories through Nexus, you can:
- **Scan for Vulnerabilities:** Use integrated security tools to scan for and identify vulnerabilities in your dependencies.
- **Control Access:** Control and verify the packages that enter your environment, ensuring that only trusted and secure dependencies are used.
- **Monitor and Audit:** Keep an audit trail of all downloaded packages, allowing you to monitor and respond to potential security issues.

### List of Maven Repositories to Proxy

To get started with proxying repositories, here’s a list of common Maven repositories that you can proxy through your Nexus server:

| Repository Name      | Repository URL                                                     | Description                                                                   |
| -------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| Apache Maven         | https://repo.maven.apache.org/maven2/                              | The central repository for most Java and Android libraries.                   |
| Google Maven         | https://maven.google.com/                                          | Google's repository for Android libraries and Google services.                |
| Google Android Maven | https://dl.google.com/dl/android/maven2/                           | Google's repository specifically for Android libraries.                       |
| JCenter              | https://jcenter.bintray.com/                                       | A popular repository that hosts a variety of Java and Android libraries.      |
| Gradle Plugin Portal | https://plugins.gradle.org/m2/                                     | Repository for Gradle plugins.                                                |
| JitPack              | https://jitpack.io/                                                | Provides access to GitHub-hosted repositories with automatic Maven packaging. |
| Sonatype Snapshots   | https://oss.sonatype.org/content/repositories/snapshots/           | Repository for snapshot versions of Sonatype projects.                        |
| Spring Plugins       | https://repo.spring.io/plugins-release/                            | Repository for Spring plugins and other related libraries.                    |
| Spring Milestones    | https://repo.spring.io/milestone/                                  | Repository for milestone versions of Spring framework libraries.              |
| Spring Snapshots     | https://repo.spring.io/snapshot/                                   | Repository for snapshot versions of Spring framework libraries.               |
| Kotlin EAP           | https://maven.pkg.jetbrains.space/kotlin/p/kotlin/eap              | Early access preview for Kotlin libraries.                                    |
| JetBrains            | https://packages.jetbrains.team/maven/p/kotlin/kotlin-dev          | Repository for JetBrains-related libraries, including Kotlin.                 |
| Atlassian            | https://packages.atlassian.com/maven/repository/public             | Repository for Atlassian-related libraries.                                   |
| Apache Snapshots     | https://repository.apache.org/snapshots/                           | Repository for Apache project snapshot versions.                              |
| JBoss Releases       | https://repository.jboss.org/nexus/content/repositories/releases/  | Repository for JBoss community releases.                                      |
| JBoss Snapshots      | https://repository.jboss.org/nexus/content/repositories/snapshots/ | Repository for JBoss community snapshots.                                     |
| RedHat GA            | https://maven.repository.redhat.com/ga/                            | Repository for RedHat supported libraries and dependencies.                   |
| RedHat EA            | https://maven.repository.redhat.com/earlyaccess/                   | Early access repository for RedHat libraries and dependencies.                |
| IBM Cloud Maven      | https://mobilefirstplatform.ibmcloud.com/maven/repo/               | Repository for IBM Cloud mobile libraries.                                    |
| Bintray              | https://dl.bintray.com/                                            | Repository for general-purpose binaries and artifacts hosted by Bintray.      |

### Automating the Creation of Proxied Repositories

To simplify the process of creating proxied repositories for the above list, you can use the following bash script. This script will create each repository in Nexus using the names and URLs provided:

```bash
#!/bin/bash

# Check if the correct number of arguments is provided
if [ "$#" -ne 2 ]; then
    echo "Usage: $0 <nexus-username> <nexus-password>"
    exit 1
fi

NEXUS_USER=$1
NEXUS_PASSWORD=$2
NEXUS_URL="http://localhost:8081" # Replace with your Nexus URL

# Define the repositories
declare -A repositories=(
    ["apache-maven"]="https://repo.maven.apache.org/maven2/"
    ["google-maven"]="https://maven.google.com/"
    ["google-android-maven"]="https://dl.google.com/dl/android/maven2/"
    ["jcenter"]="https://jcenter.bintray.com/"
    ["gradle-plugin-portal"]="https://plugins.gradle.org/m2/"
    ["jitpack"]="https://jitpack.io/"
    ["sonatype-snapshots"]="https://oss.sonatype.org/content/repositories/snapshots/"
    ["spring-plugins"]="https://repo.spring.io/plugins-release/"
    ["spring-milestones"]="https://repo.spring.io/milestone/"
    ["spring-snapshots"]="https://repo.spring.io/snapshot/"
    ["kotlin-eap"]="https://maven.pkg.jetbrains.space/kotlin/p/kotlin/eap"
    ["jetbrains"]="https://packages.jetbrains.team/maven/p/kotlin/kotlin-dev"
    ["atlassian"]="https://packages.atlassian.com/maven/repository/public"
    ["apache-snapshots"]="https://repository.apache.org/snapshots/"
    ["jboss-releases"]="https://repository.jboss.org/nexus/content/repositories/releases/"
    ["jboss-snapshots"]="https://repository.jboss.org/nexus/content/repositories/snapshots/"
    ["redhat-ga"]="https://maven.repository.redhat.com/ga/"
    ["redhat-ea"]="https://maven.repository.redhat.com/earlyaccess/"
    ["ibm-cloud-maven"]="https://mobilefirstplatform.ibmcloud.com/maven/repo/"
    ["bintray"]="https://dl.bintray.com/"
)

# Function to create a Nexus repository
create_nexus_repo() {
    local repo_name=$1
    local repo_url=$2

    curl -u "$NEXUS_USER:$NEXUS_PASSWORD" -X POST "$NEXUS_URL/service/rest/v1/repositories/maven/proxy" -H "Content-Type: application/json" -d "{
      \"name\": \"$repo_name\",
      \"online\": true,
      \"storage\": {
        \"blobStoreName\": \"default\",
        \"strictContentTypeValidation\": true,
        \"writePolicy\": \"ALLOW_ONCE\"
      },
      \"cleanup\": {
        \"policyNames\": [
          \"string\"
        ]
      },
      \"proxy\": {
        \"remoteUrl\": \"$repo_url\",
        \"contentMaxAge\": 1440,
        \"metadataMaxAge\": 1440
      },
      \"negativeCache\": {
        \"enabled\": true,
        \"timeToLive\": 1440
      },
      \"httpClient\": {
        \"blocked\": false,
        \"autoBlock\": true
      },
      \"routingRule\": \"\",
      \"maven\": {
        \"versionPolicy\": \"RELEASE\",
        \"layoutPolicy\": \"STRICT\"
      }
    }"
}

# Loop through repositories and create them in Nexus
for repo in "${!repositories[@]}"; do
    echo "Creating repository: $repo"
    create_nexus_repo "$repo" "${repositories[$repo]}"
    echo ""
done

echo "All repositories have been created."
```

### Configuring Gradle to Use Your Nexus Repositories

After setting up your Nexus server to proxy these repositories, you'll need to configure your Gradle build to use them instead of the default repositories. Below is an example of how you can configure your `build.gradle` file to point to your Nexus server.

#### Updated `build.gradle` Configuration

```groovy
repositories {
    // Clear existing repositories
    repositories.clear()

    // Define the base URL for your Nexus server
    def nexusUrl = "https://nexus.yourdomain.com/repository"

    // Add all Nexus repositories
    maven {
        name = "Apache Maven"
        url = "$nexusUrl/apache-maven/"
    }
    maven {
        name = "Google Maven"
        url = "$nexusUrl/google-maven/"
    }
    maven {
        name = "Google Android Maven"
        url = "$nexusUrl/google-android-maven/"
    }
    maven {
        name = "JCenter"
        url = "$nexusUrl/jcenter/"
    }
    maven {
        name = "Gradle Plugin Portal"
        url = "$nexusUrl/gradle-plugin-portal/"
    }
    maven {
        name = "JitPack"
        url = "$nexusUrl/jitpack/"
    }
    maven {
        name = "Sonatype Snapshots"
        url = "$nexusUrl/sonatype-snapshots/"
    }
    maven {
        name = "Spring Plugins"
        url = "$nexusUrl/spring-plugins/"
    }
    maven {
        name = "Spring Milestones"
        url = "$nexusUrl/spring-milestones/"
    }
    maven {
        name = "Spring Snapshots"
        url = "$nexusUrl/spring-snapshots/"
    }
    maven {
        name = "Kotlin EAP"
        url = "$nexusUrl/kotlin-eap/"
    }
    maven {
        name = "JetBrains"
        url = "$nexusUrl/jetbrains/"
    }
    maven {
        name = "Atlassian"
        url = "$nexusUrl/atlassian/"
    }
    maven {
        name = "Apache Snapshots"
        url = "$nexusUrl/apache-snapshots/"
    }
    maven {
        name = "JBoss Releases"
        url = "$nexusUrl/jboss-releases/"
    }
    maven {
        name = "JBoss Snapshots"
        url = "$nexusUrl/jboss-snapshots/"
    }
    maven {
        name = "RedHat GA"
        url = "$nexusUrl/redhat-ga/"
    }
    maven {
        name = "RedHat EA"
        url = "$nexusUrl/redhat-ea/"
    }
    maven {
        name = "IBM Cloud Maven"
        url = "$nexusUrl/ibm-cloud-maven/"
    }
    maven {
        name = "Bintray"
        url = "$nexusUrl/bintray/"
    }
}
```

#### Explanation:
- **nexusUrl**: This is the base URL for your Nexus server. Replace `https://nexus.yourdomain.com` with the actual URL if your Nexus server is hosted elsewhere.
- **maven { name = "..." url = "..." }**: Each block adds a repository from your Nexus server, named appropriately for clarity.

### Troubleshooting: Why Might Android Studio Still Download from `repo.maven.apache.org`?

If after following these steps, Android Studio still downloads packages from `repo.maven.apache.org`, it could be due to several factors:

- **Global Configuration Override:** Your project settings might be overridden by global Gradle settings or Android Studio's internal settings. Check your `gradle.properties` and Android Studio configurations.
  
- **Dependency Resolution Strategy:** If a dependency isn't found in your Nexus repository, Gradle might fall back to `repo.maven.apache.org`. Ensure all dependencies are correctly proxied and available in your Nexus.

- **Gradle Daemon Cache:** Gradle's daemon might cache settings. You can stop the Gradle daemon and clear the cache with:
  ```bash
  ./gradlew --stop
  ./gradlew cleanBuildCache
  ```

- **Offline Mode:** Ensure Android Studio isn’t in offline mode, which might bypass your Nexus setup.

### Conclusion

Proxying Maven repositories locally on your Nexus server provides significant advantages in terms of load times, access continuity, and security. By caching your dependencies locally, you can ensure faster builds, uninterrupted access to crucial packages, and a more secure development environment. With the provided script, setting up these proxied repositories is a straightforward process, allowing you to focus on what matters most—building great software. 

After configuring your Nexus server, be sure to update your Gradle configuration as outlined above. Should you encounter issues where dependencies are still being fetched from the original repositories, carefully review your Gradle and Android Studio configurations to ensure that everything is properly set up to use your Nexus repositories.
