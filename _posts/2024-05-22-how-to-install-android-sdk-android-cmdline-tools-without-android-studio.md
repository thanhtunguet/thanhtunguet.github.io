---
title: "How to Install Android SDK and Command-Line Tools on Ubuntu Without Android Studio"
date: 2024-05-22 02:24:34 +0700
categories: ["Mobile Development", "Android"]
tags: ["Mobile Development", "Android Studio", "Command-Line Tools"]
pin: false
---

If you're developing Android applications but want to avoid the overhead of Android Studio, you can install the Android SDK and command-line tools directly on your Ubuntu system. This guide will walk you through the steps to get everything set up.

## Prerequisites

Before you begin, ensure you have the following:

- An Ubuntu system (or any Debian-based Linux distribution).
- A terminal to run the commands.

## Step 1: Update Your Package List

First, update your package list to make sure you have the latest information on the newest versions of packages and their dependencies.

```bash
sudo apt update
```

## Step 2: Install Required Dependencies

Install `unzip` and `wget`, which are necessary to download and extract the command-line tools.

```bash
sudo apt install unzip wget
```

## Step 3: Download the Latest Command-Line Tools

Navigate to the [Android Studio download page](https://developer.android.com/studio#command-tools) to find the latest command-line tools for Linux. As of the time of writing, you can download them using the following command:

```bash
wget https://dl.google.com/android/repository/commandlinetools-linux-10406996_latest.zip -O commandlinetools.zip
```

## Step 4: Create a Directory for the Android SDK

Create a directory where the Android SDK and command-line tools will reside.

```bash
mkdir -p ~/Android/Sdk/cmdline-tools
```

## Step 5: Unzip the Downloaded Command-Line Tools

Extract the downloaded command-line tools to the directory you just created.

```bash
unzip commandlinetools.zip -d ~/Android/Sdk/cmdline-tools
```

## Step 6: Rename the Unzipped Folder

Rename the unzipped folder to `latest` (or `tools`). This step ensures that the path is consistent with what the tools expect.

```bash
mv ~/Android/Sdk/cmdline-tools/cmdline-tools ~/Android/Sdk/cmdline-tools/latest
```

## Step 7: Add the Android SDK and Command-Line Tools to Your PATH

To make the command-line tools accessible from any terminal session, you'll need to add them to your PATH. Open or create the `.bashrc` file in your home directory:

```bash
nano ~/.bashrc
```

Add the following lines to the end of the file:

```bash
export ANDROID_SDK_ROOT=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
```

Save and close the file (Ctrl+X, then Y, then Enter in nano).

## Step 8: Apply the Changes

To apply the changes to your current shell session, source the `.bashrc` file:

```bash
source ~/.bashrc
```

## Step 9: Verify the Installation

Run the following command to verify that the installation was successful:

```bash
sdkmanager --version
```

You should see the version of the SDK manager printed in the terminal.

## Step 10: Install Essential SDK Packages

Finally, install the essential SDK packages, such as `platform-tools`, specific Android platforms, and build tools. Replace `android-30` and `30.0.3` with the versions you need:

```bash
sdkmanager "platform-tools" "platforms;android-30" "build-tools;30.0.3"
```

## Conclusion

You have now successfully installed the Android SDK and command-line tools on your Ubuntu system without the need for Android Studio. This setup allows you to manage your Android development environment more efficiently and with less overhead. Happy coding!
