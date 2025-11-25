---
title: "Automating Mailcow Mailbox Creation with Selenium and TypeScript"
date: 2025-04-01 12:30:00 +0700
categories: ["DevOps", "Infrastructure"]
tags: ["DevOps", "Mailcow", "Automation"]
pin: false
mermaid: true
---

Managing a growing list of email accounts manually in [Mailcow](https://mailcow.email/) can be tedious. Unfortunately, the default UI lacks bulk import tools, and the API requires careful CSRF and session handling.

In this post, I’ll walk you through how I used **TypeScript + Node.js + Selenium WebDriver** to automate **bulk mailbox creation in Mailcow**, using real browser automation — including login, CSRF token parsing, and mailbox creation using Mailcow's internal APIs.

---

## 🧩 Problem Statement

You have an Excel sheet (or JSON file) full of email accounts you want to create in Mailcow. Mailcow doesn’t provide a native “Import Mailboxes” feature via the UI. 

The REST API exists, but requires:
- CSRF token
- Session cookies
- Non-trivial JSON encoding in form submissions

Rather than mimicking this via `curl`, we can **simulate actual browser behavior** — logging in, extracting tokens, and submitting forms like a real admin.

---

## 🔧 Tech Stack

- **Node.js** with **TypeScript**
- **Selenium WebDriver** with Chrome
- **dotenv** for environment config
- JSON file for mailbox user data
- Manual CSRF token extraction from inline JavaScript

---

## ⚙️ Setup and Configuration

First, install the dependencies:

```bash
npm install selenium-webdriver chromedriver dotenv
npm install -D typescript ts-node
```

Then, create a `.env` file to configure your Mailcow instance:

```env
MAILCOW_URL=https://mail.example.com
MAILCOW_ADMIN=admin
MAILCOW_PASSWORD=123
MAILCOW_DEFAULT_PASSWORD=UserPass123!
```

---

## 📁 Input: `users.json`

We store user data in a JSON file:

```json
[
  { "email": "john@example.com", "name": "John Nguyễn" },
  { "email": "jane@example.com", "name": "Jane Phạm" }
]
```

This gets normalized into local part/domain pairs and ASCII-safe names using a tone-removal function for Vietnamese characters.

---

## 🔍 The Magic: Selenium + In-Browser Fetch

Here’s what the script does:

### ✅ 1. Log in to Mailcow
Selenium opens the login page, fills in the username/password, and clicks the login button. Once it confirms login via URL change, we proceed.

### ✅ 2. Parse the CSRF token
Mailcow sets a global JS variable:

```js
var csrf_token = 'abc123';
```

We extract this using a regular expression from the page’s HTML source.

### ✅ 3. Submit mailbox creation via browser `fetch()`
We inject a `fetch` call into the browser using `driver.executeScript()`, passing Mailcow user data and the CSRF token.

This simulates the AJAX behavior the UI uses behind the scenes, making the process very reliable.

---

## 🧪 Developer Mode: DevTools On

To help debug or observe behavior:
- We launch Chrome with DevTools open using `--auto-open-devtools-for-tabs`
- You can visually inspect network requests and page behavior

This is great for verifying POST requests and token handling.

---

## 🧠 Why Not Just Use the API Directly?

While Mailcow does have API endpoints (`/api/v1/add/mailbox`), they:
- Expect CSRF token even with cookies
- Reject malformed encoded JSON in `x-www-form-urlencoded` format
- Are best used with full session simulation

By using Selenium and working inside the browser, **we bypass all of that complexity**.

---

## ✅ Output Example

```text
✅ Logged in
🔑 CSRF token: 5ae67dfc
📬 Request sent for: john@example.com
📬 Request sent for: jane@example.com
```

---

## 🧠 Extending the Script

Here’s what you could add next:
- 📥 Load from Excel with `xlsx`
- 🧪 Dry-run mode
- 🔁 Retry logic for failed mailbox creation
- 🔄 Create aliases or forwarding rules
- 🐳 Dockerize the script

---

## 🧾 Full Source Code

The full script is available on GitHub here: [github.com/your-org/mailcow-bulk-mailbox](#)  
(Or you can ask me for the full inline code if needed.)

---

## 💡 Conclusion

This approach gives you the **reliability of a logged-in admin session** and the **power of API automation**, without reverse-engineering Mailcow’s CSRF/session handling.

It’s an elegant, browser-based workaround for an otherwise tedious process.

---

💬 Have questions? Want to extend this into a more powerful Mailcow CLI tool or integrate it into your provisioning system? Feel free to reach out or drop a comment!
