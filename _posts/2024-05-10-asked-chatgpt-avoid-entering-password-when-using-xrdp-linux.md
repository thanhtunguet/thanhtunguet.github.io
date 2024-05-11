---
title: "Asked ChatGPT: Avoid entering password when using xRDP on Linux"
date: 2024-05-10 00:01:00 +0700
categories: [AI, "Asked ChatGPT"]
tags: [AI, "ChatGPT", "Asked ChatGPT"]
pin: false
---

To avoid the system asking for your password to create color profiles and managed devices each time you access your Ubuntu system via xrdp, you can modify the Polkit rules associated with those actions. Polkit (PolicyKit) is the system service that manages user privileges. 

Here's how you can modify the Polkit rules:

1. Open a terminal on your Ubuntu system.
2. Navigate to the Polkit rules directory by running the following command:
   ```
   cd /etc/polkit-1/localauthority/50-local.d/
   ```
3. Create a new .pkla file for your custom rules. You can name it something like `disable-color-profile.pkla`:
   ```
   sudo nano disable-color-profile.pkla
   ```
4. Add the following lines to the file:
   ```
   [Disable Color Profile]
   Identity=unix-user:YOUR_USERNAME
   Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile
   ResultAny=no
   ResultInactive=no
   ResultActive=yes
   ```
   Replace `YOUR_USERNAME` with your actual username on the Ubuntu system.

5. Save the file and exit the text editor.

6. Restart the Polkit service for the changes to take effect:
   ```
   sudo systemctl restart polkit.service
   ```

These rules will prevent the system from asking for your password when creating color profiles and managed devices via xrdp. Remember to replace `YOUR_USERNAME` with your actual username.

After applying these changes, test accessing your Ubuntu system via xrdp again to see if the password prompts are now bypassed.
