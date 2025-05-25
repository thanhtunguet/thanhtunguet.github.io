---
title: Setting Up a Development Environment for macOS for Mobile Programming (Part 1)
date: 2023-03-28 10:10:00 +0700
categories: ["Mobile Development", macOS]
tags: ["Mobile Development", macOS", "Homebrew", "ZSH", "NVM"]
pin: false
---

## Introduction

Setting up a development environment on macOS can be complex but rewarding. This guide walks you through the essential steps to prepare your macOS for mobile programming, focusing on tools like Homebrew, ZSH, and version managers.

---

## Understanding macOS Chip Architectures

### Apple Silicon (ARM)

Apple Silicon chips (e.g., M1, M2) offer high performance and energy efficiency. They are the default architecture for newer Macs.

### Intel-Core (x86-64)

Older Macs use Intel chips. Many applications still support this architecture, but performance may vary.

### How to Check Your Mac's Chip Type

1. Click the **Apple Menu** and select **About This Mac**.
2. Look for the **Processor** or **Chip** information.

---

## Installing Command-Line Tools and Homebrew

### Command-Line Tools

Command-Line Tools provide essential utilities like `git` and `curl`. Install them using:

```bash
xcode-select --install
```

### Homebrew

Homebrew is a package manager for macOS. Install it using:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

After installation, you can install packages with:

```bash
brew install <package_name>
```

---

## Enhancing the Terminal with ZSH Plugins

ZSH is the default shell on macOS. Enhance it with **Oh-My-Zsh** and plugins like `zsh-syntax-highlighting` and `zsh-autosuggestions`.

### Install Oh-My-Zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### Add Plugins

1. Clone the plugins:

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

2. Edit `~/.zshrc` to include the plugins:

```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

3. Restart the terminal or run:

```bash
source ~/.zshrc
```

---

## Managing Programming Language Versions

### Node.js with NVM

Install NVM using:

```bash
brew install nvm
```

Add the following to `~/.zshrc`:

```bash
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

#### Common NVM Commands

- Install the latest version of Node.js:

```bash
nvm install node
```

- Install a specific version:

```bash
nvm install 16
```

- Set a default version:

```bash
nvm alias default 16
```

---

## Conclusion

This guide covers the foundational tools for setting up a macOS development environment. In Part 2, we'll dive into installing Xcode, Android Studio, and Flutter.
