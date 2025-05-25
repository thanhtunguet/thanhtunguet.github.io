---
title: Setting Up a Development Environment for MacOS for Mobile Programming (Part 1)
date: 2023-03-28 10:10:00 +0700
categories: ["Mobile Development", MacOS]
tags: ["Mobile Development", MacOS]
pin: false
---

## Introduction

Setting up a development environment is not too difficult, but it requires time and complex steps to fully complete an environment that serves programming, especially cross-platform mobile programming with Flutter and React Native. This article guides and explains each installation step to help you better understand the MacOS platform, Android & iOS programming platforms with React Native or Flutter.

## Understanding Mac and MacOS

### Chip Architecture

Apple's Mac computers use different chip architectures to provide the best performance and features for users. These chip architectures include:

### Apple Silicon (ARM)

Apple Silicon is the general name for chips designed by Apple and running on the latest Mac computers. These chips have higher computational capabilities and lower power consumption compared to previous Intel chips. Apple transitioned from using Intel chips to Apple Silicon starting in 2020.

### Intel-Core (x86-64 or amd64)

Before transitioning to Apple Silicon, Mac computers used Intel chips with x86-64 architecture. These Intel chips provide the best performance and features for applications that require a lot of computational power, such as graphics and filmmaking.

### Knowing Your Machine's Chip Type

Since 2020, the market has maintained two lines of Mac machines: Mac Intel and Mac M-Series, corresponding to two architectures: x86-64 and ARM. Since 2021, most software has been updated to support the Mac M line with the following options:

- A separate build for each architecture

![picture 11](/assets/img/1bf577b6d241c9878e0fcd561bdced5ba5830e6e3eb72f14bc916ad788c785bd.png)  

- Universal build

![picture 13](/assets/img/cfde445227764c9e27c09433e3e5f91fe3a3e16944a27e533cf17ada750ece9f.png)  

You need to know your machine's chip type to choose the corresponding build for the software you use. To check the chip type, from the Apple Menu (Apple logo), select **About this Mac**:

![picture 10](/assets/img/ba8094b2008eee21b821bb7175442f27a4458915e43467db3c8999d88d6ce386.png)  

## Installing Command-line Tools and Homebrew

Command-line tools and Homebrew are foundational tools for installing other packages, so they need to be done first.

### Command-line Tools

`Command-line tools` is a toolkit that provides users with command-line tools for developing applications on macOS. Developed by Apple, `command-line tools` provide users with package management tools, source code (Git), compilation and application building tools, and many other tools to help users develop applications on macOS.

Most programming environments on MacOS require this package. Additionally, we need to use `git` and `curl` to install other packages, including Homebrew. Therefore, we need to install Command-line tools first when setting up the environment for our Mac.

To use `Command-line tools`, users need to install XCode on their macOS. After installing XCode, users can install XCode `command-line tools` through the Terminal on macOS using the command:

```
xcode-select --install

```

Installing `command-line tools` will allow users to use command-line tools to develop applications on macOS.

### Homebrew

Homebrew is a package manager for MacOS, helping you install applications and libraries easily through the Terminal. To install Homebrew, you can use the following command in the Terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

```

After installation, you can use the following command to install packages on Homebrew:

```bash
brew install {package_name}
```

For example, to install Chrome on Homebrew, you can use the command:

```bash
brew install google-chrome

```

Additionally, you can search for packages on Homebrew using the command:

```bash
brew search {package_name}
```

Note: You need to install Command-line tools first because CLT already includes curl for the Homebrew installation command below.

## Installing ZSH Plugins

ZSH is a shell (command-line environment) developed to replace Bash, commonly used on operating systems like Linux and MacOS. ZSH has more advanced features than Bash, including automatic command completion and easy configuration.

Oh-my-zsh is a plugin for ZSH, providing users with additional features and an easy-to-use interface. Oh-my-zsh includes various themes to change the terminal's appearance and over 150 different plugins to provide different features for ZSH.

ZSH is currently the default Shell for MacOS Terminal.

This step is not mandatory. However, I encourage you to install it for a more useful and user-friendly terminal, with two plugins `zsh-syntax-highlighting` and `zsh-autosuggestions`, you will find it much easier to use the Terminal as it supports command highlighting and suggests available or frequently used commands based on your command history.

For example, with the command `adb reverse tcp:8090 tcp:8090`, which I frequently use to open Objectbox Admin when programming Flutter. When fully installed, the plugin will support automatic command completion like this:

![picture 6](/assets/images/b45a08485a797ba8ae0742ac59e160bda7cded41c34c135c27dc9705f3b6c166.png)

### Installing Oh-my-zsh

Use the following command to install `Oh-my-zsh`:

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

After installation, `oh-my-zsh` will automatically open a new session in the current terminal. You can keep this session to continue installing the next two plugins or exit and open a new terminal.

### Installing zsh-syntax-highlighting and zsh-autosuggestions

Step 1: Use Git to clone the two plugins

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

Step 2: Configure `oh-my-zsh` to automatically load these two plugins whenever you open a new terminal:

- Open the file `~/.zshrc`

  ```bash
  nano ~/.zshrc
  ```

- Find the section:

  ```bash
  plugins=(git)
  ```

- And change it to:
  ```bash
  plugins=(
    git
    zsh-autosuggestions
    zsh-syntax-highlighting
  )
  ```

Step 3: Restart the terminal

Or enter the command:

```bash
source ~/.zshrc
```

### Important Note

When installing `oh-my-zsh`, this plugin will overwrite the `~/.zshrc` file, which is the configuration file for the terminal's zsh.
Therefore, this step should also be done before installing other environments.

Reference link: [https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/](https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/)

## Version Manager for Popular Programming Languages

A common trend among programming languages recently is the `version manager`. Popular programming languages often have new versions periodically and frequently add features/changes. These changes are often made faster than libraries/frameworks. Therefore, each framework, library, or project often requires different versions for each language.

For example:

With React Native, from version 0.70 and earlier, it can run with Node.js `v16`, but from 0.71, it will require Node.js `v18`.

Therefore, when setting up a programming environment, I encourage you to install all the corresponding version manager tools.

### NVM

Let's start with `NVM` for Node.js

```bash
brew install nvm
```

Add the following to the end of the `~/.zshrc` file:

```bash
# Node version manager
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

After installing `NVM`, you will need the following commands:

#### Install the latest Major version:

```bash
nvm install 16
```

NVM will install the latest version of `v16`

- Install a specific `major.minor.patch` version

```bash
nvm install 16.15
```

These versions follow the `semantic versioning` format.

#### Switch versions for the current shell session:

```bash
nvm use 16
```

or

```bash
nvm use 16.15
```

Note: This command requires specifying a previously installed version.

#### Set the default version for Node.js

This step is extremely important for React Native or Web front-end developers.

Since Xcode or IDEs will use the default shell configuration to execute code bundle commands, they will use the default specified version of Node.js

```bash
nvm alias default <version>
```
