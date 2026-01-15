# Android Web Server using Termux & Cloudflare Tunnel

## Overview

- Host a website on an Android device
- Use Termux as a Linux environment
- Access Termux remotely using SSH (MobaXterm)
- Run Apache as a background Linux service
- Expose the site publicly using Cloudflare Quick Tunnel
- No domain or public IP required
- Server continues running after closing Termux

---

## Step 0: Enable SSH Access (Termux ➜ MobaXterm)

### Step 1: Install OpenSSH

```bash
pkg install openssh -y
````

---

### Step 2: Set Password

```bash
passwd
```

---

### Step 3: Start SSH Server

```bash
sshd
```

* Default SSH port: **8022**

---

### Step 4: Find Termux Local IP

```bash
ip addr | grep inet
```

Example:

```text
192.168.1.5
```

---

### Step 5: Connect from MobaXterm

* Open **MobaXterm**
* Create **New SSH Session**
* Use:

  * Host: `TERMUX_IP`
  * Port: `8022`
  * Username: Termux username
* Enter password when prompted

---

## Step 6: Update System Packages

```bash
pkg update && pkg upgrade -y
```

---

## Step 7: Install Required Services

```bash
pkg install apache2 runit cloudflared nano curl wget -y
```

---

## Step 8: Start Apache Web Server

```bash
apachectl start
```

* Apache runs on **port 8080**

Test locally:

```bash
curl http://127.0.0.1:8080
```

---

## Step 9: Create Website Files

Website directory:

```text
$PREFIX/share/apache2/default-site/htdocs
```

Create index file:

```bash
nano $PREFIX/share/apache2/default-site/htdocs/index.html
```

---

## Step 10: Website HTML Code

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Android Web Server</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
      color: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      text-align: center;
    }
    .card {
      background: rgba(0, 0, 0, 0.4);
      padding: 40px;
      border-radius: 12px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);
      max-width: 420px;
    }
    h1 {
      margin-bottom: 10px;
    }
    p {
      opacity: 0.9;
    }
    .badge {
      margin-top: 20px;
      display: inline-block;
      padding: 8px 16px;
      border-radius: 20px;
      background: #00c6ff;
      color: #000;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🚀 Android Web Server</h1>
    <p>This website is hosted on an Android device using Termux and Apache.</p>
    <p>Public access is provided via Cloudflare Tunnel.</p>
    <div class="badge">Live from Termux</div>
  </div>
</body>
</html>
```

---

## Step 11: Start Cloudflare Quick Tunnel

```bash
cloudflared tunnel --url http://localhost:8080
```

* A public HTTPS URL will be generated:

```text
https://xxxxx.trycloudflare.com
```

---

## Step 12: Run Web Server as a Linux Service

### Step 13: Create Service Directory

```bash
mkdir -p $PREFIX/var/service/start_site
```

---

### Step 14: Create Service Run Script

```bash
nano $PREFIX/var/service/start_site/run
```

```bash
#!/data/data/com.termux/files/usr/bin/bash

apachectl start

cloudflared tunnel --url http://localhost:8080 \
  --logfile $PREFIX/var/log/cloudflared.log \
  --loglevel info
```

---

### Step 15: Enable and Start Service

```bash
chmod +x $PREFIX/var/service/start_site/run
sv-enable start_site
sv up start_site
```

---

## Step 16: Get Public Website URL

```bash
tail -f $PREFIX/var/log/cloudflared.log
```

Look for:

```text
https://xxxxx.trycloudflare.com
```

---

## Notes

* Cloudflare Quick Tunnel URL changes after restart
* HTTPS is enabled automatically
* Disable battery optimization for Termux
* Closing Termux will NOT stop the server

---

## Author

* **Khizer Wasim**
