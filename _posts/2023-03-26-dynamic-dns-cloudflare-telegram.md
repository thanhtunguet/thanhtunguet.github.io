---
title: Solving Dynamic IP Issues with Cloudflare API and Telegram Bot
date: 2023-03-26 19:10:00 +0700
categories: [Devops, Networks]
tags: [Devops, DNS, Networks, Cloudflare, "Scripts"]
pin: false
---

## Problem Statement

Are you a student needing to complete a major project without spending too much on cloud services?
Do you want to set up a web server at home using an old computer as a server?
Do you have an IP camera or other IoT devices and want to access them remotely?
And many other needs.

Clearly, at home, you have an internet connection you use daily, and old desktop or laptop computers you no longer use. Can we utilize them to set up a server accessible from the internet? What issues might arise? In this article, I'll help you explore and guide you on how to make use of what's "available" to meet your needs. Additionally, I hope you gain some network knowledge if you're not already familiar.

Moreover, this article guides you on utilizing a home internet connection (with dynamic IP), so if you already have a static IP, feel free to skip this article.

## Preparation

To start, you need to prepare the following:

- A stable internet connection with access to the main router to configure port forwarding. This is relatively easy if you're at your own home but might be challenging if you're renting, as rental places often have at least two network layers, and you don't have access to the main router.
- A computer, of course.
- A domain name. You can register for free at [https://www.freenom.com/](https://www.freenom.com/) or purchase one if you can afford it.
- Other computers/internet devices you need.

Note:
- Some ISPs, like FPT Telecom, do not open ports by default, and you need to call their support to request it.

## Dynamic IP Issue

Typically, home internet packages are configured with dynamic IPs to save costs, as IPs are a finite resource. You can read more [here](https://wiki.matbao.net/ip-la-gi-tong-hop-moi-kien-thuc-can-biet-ve-dia-chi-ip/).

And that's the problem. Because the IP address is dynamic, it frequently changes, even within a day, leading to situations where you've checked and noted your home's IP address, but it changes as soon as you step out, and you can't access your home network anymore.

## Solution

The solution is quite simple. There are two ways to address this:

### Buy a Static IP

Quick, convenient, saves time, but costs twice as much. :)) If you choose this method, you don't need to read further. But since we want to utilize what's available, save money, and learn more about networking, let's move to the second solution.

### Attach a Fixed Domain and Update IP for the Domain Whenever It Changes

Use a script to continuously check the IP address and update it, notifying you whenever it changes. Here, I use Cloudflare and Telegram.

This way, I can access my home network through the domain I've set up. The bot will automatically check the IP if it changes and update it in Cloudflare's DNS to ensure the domain always points to my home network's IP.

### What is Cloudflare?

Cloudflare is a company providing web security and speed optimization services. It was founded in 2009 and is now one of the leading companies in this field.

Cloudflare offers many services, including DDoS protection, web application firewall, spam filtering, antivirus software, website speed optimization through content distribution (CDN), and DNS security improvement. Additionally, Cloudflare provides website management and monitoring tools, making it easy for users to manage and control their websites.

Here, one of Cloudflare's most useful features is its extremely fast DNS update capability, faster than most domestic DNS, allowing us to quickly update our IP address for the domain.

### Telegram

Telegram is a messaging app that most people know; it's fast, stable, and supports easy bot setup. We can set up a bot to send messages to our phone, notifying us whenever the IP address of our home network changes.

## Detailed Steps

Below are the detailed steps:

### Create a Cloudflare API Token

To create a Cloudflare API Token with DNS edit permissions, follow these steps:

Step 1: Log in to your Cloudflare account.

Step 2: Select "API Tokens" from the left menu.

Step 3: Click "Create Token".

Step 4: Choose "Edit zone DNS" in the "Permission" section.

Step 5: Enter a name for the API Token.

Step 6: To make your API Token work on all your domains, select "All zones" in the "Zone Resources" section. If you only want the API Token to work on specific domains, choose "Selected zones" and select the domains you want.

Step 7: Click the "Create" button.

Step 8: Your API Token will be created. Store this API Token in a safe place as it will not be displayed again.

After creating the API Token, you can use it to configure DNS for the script to automatically update your IP address.

### Create a Subdomain and Point to the Current IP

To check your current home IP address, execute the following command:

```bash
echo $(curl -s https://checkip.amazonaws.com)
```

Create an `A` record in Cloudflare pointing to the current IP address.

### Learn About ZONE_ID and RECORD_ID in Cloudflare DNS

Zone ID and Record ID are two parameters used in Cloudflare's DNS system to identify domains and DNS records.

Zone ID is a unique string used to identify a domain in Cloudflare's DNS system. Each domain in your Cloudflare account will have its own Zone ID. Zone ID is used to determine the scope of DNS operations performed on that domain, such as adding, deleting, or editing DNS records.

Record ID is a unique string used to identify a specific DNS record in Cloudflare's DNS system. Each DNS record for a domain will have its own Record ID. Record ID is used to identify DNS records that need to be changed, such as updating the value of an A record or adding a new record.

Obtaining the Zone ID and Record ID is crucial for managing a domain's DNS on Cloudflare's system. They are used to perform operations like updating DNS records, creating filters, and reporting on Cloudflare's DNS system.

### Obtain ZONE_ID and RECORD_ID for a DNS Record

To obtain the Zone ID and Record ID in Cloudflare DNS, follow these steps:

Step 1: Log in to your Cloudflare account.

Step 2: Select the domain for which you want to obtain the Zone ID or Record ID.

Step 3: Click on the "DNS" tab.

Step 4: Here, you will see all the DNS records for your domain.

Step 5: To obtain the Zone ID, you can view it at the top of the "Overview" page. It will be displayed next to your domain name.

Step 6: To obtain the Record ID, you can click the "Edit" icon to the right of the DNS record you want to obtain the Record ID for. A dialog box will appear, and the Record ID will be displayed at the end of the URL in your browser. For example: https://dash.cloudflare.com/xxxxxxxxxxxxxxxxxxxxxxxxxx/edit/xxxxxxxxxxxxxxxxxxxxxxxxxx.

### Create a Bot for Telegram

Step 1: Search for the "BotFather" account in Telegram and start a conversation with it.

Step 2: Send the message "/newbot" to start creating a new Bot.

Step 3: Enter a name for your Bot.

Step 4: Then, enter a username for your Bot. Note that the username must end with "bot", for example: "mytelegrambot_bot".

Step 5: If everything goes smoothly, BotFather will send you an access token. Save this token, as you will need it to connect to your Bot.

Step 6: Now, you have successfully created a Bot in Telegram. You can use your access token to connect to your Bot through Telegram's API. You can use programming libraries like Telebot, pyTelegramBotAPI, or python-telegram-bot to program your Bot.

### Obtain the Chat ID of a Telegram Account

The Chat ID of a Telegram account is a unique integer used to identify that account. Chat ID is used to interact with Bots or Telegram API, such as sending messages to a specific account, retrieving account information, or performing other activities.

To obtain the Chat ID of your current Telegram account, follow these steps:

Step 1: Open the Telegram app and search for the "@userinfobot" account.

Step 2: Start a conversation with the "@userinfobot" account.

Step 3: Send the message "/start" to start using the bot.

Step 4: The bot will send you a message containing information about your account, including the Chat ID.

Step 5: Copy the Chat ID for programming purposes or interacting with Telegram API.

### Write the Complete Script

Now that we've set up the DNS record and Telegram bot, we'll write a complete BASH script to perform the following tasks:

- Check the current public IP address of the home network
- Check the IP address on Cloudflare's DNS record
- Compare the two IPs. If they differ, it means the network's IP has changed. In this case, we'll send a notification via the bot to Telegram and update the IP address on Cloudflare.

```bash
#!/bin/bash

# Cloudflare Configuration
CF_API_KEY="CLOUFLARE_API_KEY"  # Your Cloudflare API key
CF_EMAIL="your_email@example.com"  # Your Cloudflare account email
ZONE_ID="ZONE_ID"  # Zone ID for your domain
RECORD_ID="RECORD_ID"  # Record ID for the DNS entry

# Telegram Bot Configuration
BOT_TOKEN="Bot_token"  # Telegram bot token
CHAT_ID="Your chat id"  # Telegram chat ID for notifications

# DNS Configuration
YOUR_DOMAIN="*.example.com"  # Your domain or subdomain
DNS_SERVER="8.8.8.8"  # DNS server to resolve the domain

# Function to send a message via Telegram
function sendChat() {
    curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
         -d "chat_id=${CHAT_ID}" \
         -d "text=$1"
}

# Function to call Cloudflare API
function cloudflare() {
  if [ -z "$CF_EMAIL" ]; then
      curl -sSL \
      -H "Accept: application/json" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $CF_API_KEY" \
      "$@"
  else
      curl -sSL \
      -H "Accept: application/json" \
      -H "Content-Type: application/json" \
      -H "X-Auth-Email: $CF_EMAIL" \
      -H "X-Auth-Key: $CF_API_KEY" \
      "$@"
  fi
}

# Get current public IP address of your network
PUBLIC_IP=$(curl -s https://checkip.amazonaws.com)

# Get current DNS setting of your domain
CURRENT_IP=$(nslookup "${YOUR_DOMAIN}" ${DNS_SERVER} | awk '/^Address: / { print $2 }')

# Compare the two IP addresses
if [ "$CURRENT_IP" != "$PUBLIC_IP" ]; then
    # If the IP address has changed, update DNS and send a notification
    sendChat "The public IP address has changed from: ${CURRENT_IP} to ${PUBLIC_IP}. DNS setting has been updated!"
    cloudflare -X PUT \
      "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
      --data "{\"type\":\"A\",\"name\":\"*\",\"content\":\"${PUBLIC_IP}\",\"ttl\":1,\"proxied\":false}"
else
    # If the DNS setting is up to date, do nothing
    echo "Current DNS setting is up to date"
fi
```

Note that you need to replace the values `ZONE_ID`, `RECORD_ID`, `CF_API_KEY`, `CF_EMAIL`, `BOT_TOKEN`, and `CHAT_ID` with the corresponding values generated from the above steps.

### Set Up a Cronjob

We have a complete script, now set up a cronjob to execute the script every minute, ensuring the DNS is always updated as quickly as possible.

Step 1: Save the script as `/usr/local/bin/cloudflare-check` and grant execution permission:

```bash
chmod a+x /usr/local/bin/cloudflare-check
```

Step 2, set up crontab with the command `crontab -e`

```bash
*/5 * * * * /usr/local/bin/cloudflare-check >/dev/null 2>&1
```

## Conclusion

With Cloudflare DNS and a Telegram bot, we can write a script (bot) to check the IP address and update it to a fixed domain, allowing us to easily access the home/internal network from outside.
