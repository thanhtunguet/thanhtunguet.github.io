---
title: "Create Linux desktop entry for LM Studio"
date: 2024-05-10 00:01:00 +0700
categories: [Devops, "Linux"]
tags: ["Devops", "Linux"]
pin: false
---

## Introduction

LM Studio's absence of a Linux desktop entry necessitates cumbersome manual methods for launching the application. This article explores the significance of desktop entries and how creating one for LM Studio can enhance efficiency and usability. By simplifying access and promoting consistency, desktop entries offer a seamless experience for Linux users, optimizing their workflow and productivity.

To create a desktop entry for LM Studio at `/usr/local/bin/lmstudio`, you'll need to create a `.desktop` file. Here's a sample entry you can use:

```plaintext
[Desktop Entry]
Name=LM Studio
Comment=Your comment here
Exec=/usr/local/bin/lmstudio
Icon=/path/to/icon/icon.png
Terminal=false
Type=Application
Categories=Development;IDE;
```

Replace `/path/to/icon/icon.png` with the actual path to the icon file for LM Studio. Also, don't forget to set the appropriate permissions for the file to make it executable. You can do this with the following command:

```bash
sudo chmod +x /usr/local/bin/lmstudio.desktop
```

Once you've created the file and set the permissions, LM Studio should appear in your desktop environment's application menu.

Also, here is LM Studio icon:

![lmstudio.webp](/assets/img/lmstudio.webp)
