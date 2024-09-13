---
title: Automating SSL Certificate Renewal and Reload for Azure DevOps Server on Windows Using Let's Encrypt
date: 2024-09-13 07:10:00 +0700
categories: [Devops, Azure Devops]
tags: [Devops, Azure Devops, Scripts]
pin: false
---

Keeping your SSL certificates up to date is crucial for ensuring secure communication between clients and servers. If you're running Azure DevOps Server on a Windows environment and using Let's Encrypt for SSL certificates, you may wonder how to automate the certificate renewal and reload process. Unlike Linux, where you can use `crontab` for automation, Windows requires a different approach. This guide will walk you through automating the renewal and reloading of SSL certificates for Azure DevOps Server on Windows using Win-ACME.

### Step 1: Install Win-ACME for Windows

Win-ACME is a simple, easy-to-use ACME client for Windows, designed to interact with the Let's Encrypt service. It supports automatic renewal of certificates and can be configured to trigger custom actions upon successful renewal.

1. **Download Win-ACME**:
   - Visit the [Win-ACME GitHub page](https://github.com/win-acme/win-acme/releases) and download the latest release for Windows.
   - Extract the downloaded ZIP file to a directory on your server, such as `C:\win-acme`.

2. **Run Win-ACME to Request an SSL Certificate**:
   Open a Command Prompt with administrative privileges, navigate to the Win-ACME directory, and run the following command:

   ```sh
   wacs.exe --target manual --host your-domain.com --validation http-01 --installationsiteid "Azure DevOps Server" --store centralssl
   ```

   Replace `your-domain.com` with your Azure DevOps Server domain name. This command will guide you through the steps to obtain and install the SSL certificate for your server.

### Step 2: Configure Automatic Renewal

When you run Win-ACME, it automatically sets up a scheduled task in Windows Task Scheduler to renew the SSL certificate periodically. However, you'll need to add a custom action to reload the Azure DevOps Server whenever the certificate is renewed.

1. **Open Windows Task Scheduler**:
   - Press `Win + R`, type `taskschd.msc`, and hit **Enter** to open Task Scheduler.

2. **Locate the Scheduled Task**:
   - Find the task created by Win-ACME. It is usually named something like `wacs.exe` or `Win-ACME`.
   - Right-click on the task and select **Properties**.

3. **Add a New Action to Reload Azure DevOps Server**:
   - Go to the **Actions** tab and click **New**.
   - Set **Action** to **Start a Program**.
   - In the **Program/script** field, enter `powershell.exe`.
   - In the **Add arguments (optional)** field, enter `-File "C:\Path\To\ReloadAzureDevOpsServer.ps1"`.
   - Make sure the **Start in (optional)** field is set to the directory where your PowerShell script is located.

### Step 3: Create a PowerShell Script to Reload Azure DevOps Server

You need a script that will stop and start the Azure DevOps Server services to apply the new SSL certificate. Here's a simple PowerShell script to accomplish this:

1. **Create the PowerShell Script**:
   - Open a text editor (like Notepad) and paste the following content:

   ```powershell
   # Stop Azure DevOps Server services
   Stop-Service -Name "tfsjobagent" -Force
   Stop-Service -Name "tfsserver" -Force

   # Start Azure DevOps Server services
   Start-Service -Name "tfsjobagent"
   Start-Service -Name "tfsserver"

   Write-Output "Azure DevOps Server has been reloaded with the new SSL certificate."
   ```

   - Save the file as `ReloadAzureDevOpsServer.ps1` in a directory of your choice, such as `C:\Scripts`.

   > **Note:** Replace `"tfsjobagent"` and `"tfsserver"` with the actual service names for your Azure DevOps Server instance. You can find the exact names by running `Get-Service` in PowerShell.

2. **Set Execution Policy**:
   Ensure that the script can be executed by setting the appropriate execution policy:

   ```sh
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

### Step 4: Test the Renewal and Reload Process

Before relying on the scheduled task for automatic renewal and reload, it's crucial to test the entire process:

1. **Manually Run the Scheduled Task**:
   - In Task Scheduler, right-click the Win-ACME renewal task and select **Run** to manually trigger the task.
   - Observe the output to ensure that the SSL certificate renewal is successful.

2. **Check Azure DevOps Server**:
   - Verify that the Azure DevOps Server services have been restarted and the new SSL certificate is correctly applied.
   - Open the Azure DevOps Server URL in a browser to confirm that the new certificate is being used.

### Step 5: Verify Renewal Logs and Set Up Notifications

To ensure that everything is working smoothly, monitor the renewal logs:

- Win-ACME logs its actions in a log file located in its installation directory.
- Regularly check the logs to ensure the renewal process works correctly.
- You may also consider setting up email notifications for task failures in Task Scheduler.

### Additional Considerations

- **Firewall and Permissions**: Ensure that your server's firewall and permissions allow the Let's Encrypt validation challenge (`http-01` or `dns-01`) to complete successfully.
- **Backup Configuration**: Always back up your current SSL certificates and Azure DevOps Server configuration before making any changes.
- **Service Names**: The exact service names may vary depending on your Azure DevOps Server configuration. Ensure you use the correct names in your PowerShell script.

### Conclusion

By using Win-ACME and PowerShell, you can automate SSL certificate renewal and service reload for Azure DevOps Server running on Windows. This process ensures that your server maintains secure communication channels without manual intervention, saving time and reducing the risk of expired certificates.

By setting this up, you maintain a secure environment and simplify certificate management for your Azure DevOps Server, keeping it running smoothly with minimal effort.

