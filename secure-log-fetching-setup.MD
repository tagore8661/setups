# Secure Log Fetching Setup

This guide provides a secure method for fetching production logs to a development server without giving developers direct access to the production environment.

## Overview

This setup uses:
- A dedicated `log-user` account on production
- SSH key authentication with restricted access
- `rrsync` for read-only file synchronization
- Automated log fetching scripts

---

## STEP 1 - Create Log User on Production Server

Login to **Production Server**:

```bash
sudo useradd -m log-user
sudo passwd -l log-user
```

This creates a user but **disables password login**.

---

## STEP 2 - Give Access Only to Log Folder

Your logs are here:
```
/path/to/logs/
├── xyz.log
└── abc.log
```

You want:
```
/var/log/app/
├── xyz.log
└── abc.log
```

Create a log directory for rsync:

```bash
sudo mkdir -p /var/log/app

# Create Symlinks Only for Required Files
sudo ln /path/to/logs/xyz.log /var/log/app/xyz.log
sudo ln /path/to/logs/abc.log /var/log/app/abc.log

# Now check:
ls -l /var/log/app
```

You should see:
```
xyz.log -> /path/to/logs/xyz.log
abc.log -> /path/to/logs/abc.log
```

Set permissions for log-user:

```bash
sudo chown -R log-user:log-user /var/log/app
sudo chmod 755 /var/log/app

# But also ensure original folder readable:
sudo setfacl -m u:log-user:x /path/to/logs
```

---

## STEP 3 - Generate SSH Key on Dev Server

Login to **Dev Server**:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_prod_logs -N ""
```

This creates:
```
~/.ssh/id_prod_logs
~/.ssh/id_prod_logs.pub
```

---

## STEP 4 - Copy Public Key to Production

Manually append the public key to production:

```bash
# On production server
sudo su - log-user

# Then create SSH directory:
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Create authorized_keys file:
vi ~/.ssh/authorized_keys
# Paste your public key from Dev server here.

# Then set permissions:
chmod 600 ~/.ssh/authorized_keys
```

---

## STEP 5 - Install rrsync on Production

On **Production Server**:

Check if rrsync exists:
```bash
ls /usr/share/doc/rsync/scripts/rrsync
```

If exists:
```bash
sudo cp /usr/share/doc/rsync/scripts/rrsync /usr/local/bin/
sudo chmod +x /usr/local/bin/rrsync
```

If not, install rsync and find rrsync:
```bash
sudo dnf install rsync -y
sudo find /usr -name rrsync
```

---

## STEP 6 - Restrict SSH Key (VERY IMPORTANT SECURITY STEP)

Edit this file on **Production Server**:
```bash
nano /home/log-user/.ssh/authorized_keys
```

Replace the key line with this:
```bash
command="/usr/local/bin/rrsync -ro /var/log/app/",restrict ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...yourkey... dev-log-fetcher
```

This means:

This key can:
- ✔ Only run rsync
- ✔ Read only
- ✔ Only /var/log/app folder
- ✘ No SSH shell
- ✘ No sudo
- ✘ No port forwarding
- ✘ No file upload

This is very secure.

---

## STEP 7 - Create Fetch Script on Dev Server

On **Dev Server**:

```bash
sudo nano /usr/local/bin/fetch-logs
```

Paste:
```bash
#!/bin/bash

PROD_USER="log-user"
PROD_IP="PROD-SERVER-IP"
LOCAL_DEST="$HOME/debug-logs/"
KEY_PATH="$HOME/.ssh/id_prod_logs"

mkdir -p "$LOCAL_DEST"

rsync -azP -e "ssh -i $KEY_PATH" "$PROD_USER@$PROD_IP:." "$LOCAL_DEST"

if [ $? -eq 0 ]; then
    echo "Successfully synced logs to $LOCAL_DEST"
else
    echo "Sync failed. Check permissions or network."
fi
```

Make executable:
```bash
chmod +x /usr/local/bin/fetch-logs
```

---

## STEP 8 - Developers Use This Command

Now developers just run on **Dev Server**:
```bash
fetch-logs
```

Logs will appear in **Dev Server**:
```
~/debug-logs/
├── xyz.log
└── abc.log
```

Developers can read logs from Dev server - no prod access needed.

---

## Production-Level Improvements (Important)

### Better setup:

#### Auto Sync Every 5 Minutes

On **Dev Server**:
```bash
crontab -e
```

Add:
```bash
*/5 * * * * /usr/local/bin/fetch-logs
```

Now logs auto sync every 5 minutes.
