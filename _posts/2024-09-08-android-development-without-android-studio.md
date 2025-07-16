---
title: "How to Configure a Linux Environment for Android CI/CD Without Android Studio"
date: 2024-09-08 12:30:00 +0700
categories: ["Software Development", "Mobile"]
tags: ["Mobile Development", "Android", "Linux"]
pin: false
mermaid: true
---

### Setting Up a Linux Server Environment for Android Builds Without Android Studio

In this guide, we'll walk through setting up a Linux server environment for building Android applications using Continuous Integration/Continuous Deployment (CI/CD) pipelines, without relying on the full Android Studio installation. This approach allows for a more streamlined, lightweight setup ideal for server environments.

### Step 1: Install the Android SDK

Begin by installing the Android SDK using `apt`:

```bash
sudo apt-get update
sudo apt-get install android-sdk
```

This command will install the Android SDK in the default directory (`/usr/lib/android-sdk`), which is required for building and managing Android projects.

### Step 2: Install OpenJDK 11 and 17

Android development requires a Java Development Kit (JDK). Depending on your project's requirements, you may need either OpenJDK 11 or OpenJDK 17. You can install both using the following commands:

```bash
sudo apt-get install openjdk-11-jdk
sudo apt-get install openjdk-17-jdk
```

Ensure that the correct JDK version is selected using the `update-alternatives` tool:

```bash
sudo update-alternatives --config java
```

### Step 3: Install Android Command-Line Tools

Download the Android command-line tools from the [Android Studio download page](https://developer.android.com/studio#command-tools). Choose the package for your Linux environment.

Once downloaded, extract the tools:

```bash
mkdir -p ~/android-sdk/cmdline-tools
unzip commandlinetools-linux-*.zip -d ~/android-sdk/cmdline-tools
```

Move the extracted files to the Android SDK directory:

```bash
sudo mv ~/android-sdk/cmdline-tools /usr/lib/android-sdk/
```

### Step 4: Set Environment Variables

To make the Android SDK and command-line tools accessible globally, set the environment variables `ANDROID_HOME` and `ANDROID_SDK_ROOT`. Note that `ANDROID_HOME` is deprecated, but some older tools may still reference it. The correct variable to use is `ANDROID_SDK_ROOT`.

Add the following lines to your shell configuration file (e.g., `~/.bashrc` or `~/.zshrc`):

```bash
export ANDROID_HOME=/usr/lib/android-sdk
export ANDROID_SDK_ROOT=/usr/lib/android-sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
```

Apply the changes:

```bash
source ~/.bashrc
```

### Step 5: Modify `PATH` Environment for CLI Tools

Ensure that the environment's `PATH` is correctly set to include tools like `adb`, `sdkmanager`, `aapt`, `jarsigner`, and others. This is done by including both platform tools and command-line tools from the SDK:

```bash
export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
export PATH=$PATH:$ANDROID_SDK_ROOT/build-tools
export PATH=$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin
```

### Step 6: Grant Permission for SDK Manager

By default, only the root user has permission to install or update build tools using `sdkmanager`. To allow non-root (sudo) users to manage the SDK, adjust the permissions of the Android SDK directory:

```bash
sudo chown -R $(whoami) /usr/lib/android-sdk
sudo chmod -R 777 /usr/lib/android-sdk
```

This command grants all users read, write, and execute permissions. While this approach simplifies permissions, it's less secure. For a more secure setup, grant specific permissions to the users who need them.

### Conclusion

By following these steps, you have successfully set up a Linux server environment for building Android applications without using Android Studio. This lightweight approach is ideal for CI/CD pipelines, allowing for more efficient builds in server environments.
