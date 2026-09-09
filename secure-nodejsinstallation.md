# Guide: Secure Node.js Installation and Deployment on Linux

This guide walks you through installing Node.js securely on a Linux server (Ubuntu/Debian) and configuring it to run safely under a non-interactive system user.

---

## 🛑 Prerequisites
Before starting, ensure your system package manager is fully up to date:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🛠️ Step 1: Install Node.js Securely via NodeSource

Avoid using the default repository version of Node.js, as it is often severely outdated. Instead, use the official NodeSource binaries to install a specific Long Term Support (LTS) release.

```bash
# 1. Download and run the NodeSource setup script (Example: Node.js 22 LTS)
curl -fsSL https://nodesource.com | sudo -E bash -

# 2. Install Node.js and build essentials (needed for compiled npm packages)
sudo apt-get install -y nodejs build-essential
```

Verify the installation succeeded and note down the exact binary execution path:
```bash
node -v
npm -v
which node # Usually outputs: /usr/bin/node
```

---

## 🛡️ Step 2: Create a Dedicated Non-Interactive User

Never run your Node.js application as `root`. Create a restricted system user that has no interactive login capabilities.

```bash
sudo useradd -r -s /bin/false nodeuser
```
*   `-r`: Defines this as a system user account.
*   `-s /bin/false`: Rejects all interactive SSH or local terminal logins.

---

## 📂 Step 3: Deploy Application and Lock Down Permissions

Move your project files into a production-appropriate web directory and give ownership exclusively to your new non-interactive user.

```bash
# 1. Create the application directory
sudo mkdir -p /var/www/my-node-app

# 2. Copy or clone your code into that directory
# (Ensure your package.json, server.js, etc., are in this folder)

# 3. Change ownership to the non-interactive user
sudo chown -R nodeuser:nodeuser /var/www/my-node-app

# 4. Restrict directory permissions so other unprivileged system users cannot view your code
sudo chmod 750 /var/www/my-node-app
```

Install your project dependencies as the `nodeuser` to prevent file permission mismatches:
```bash
cd /var/www/my-node-app
sudo -u nodeuser npm install --production
```

---

## ⚙️ Step 4: Configure systemd to Manage the Process

Create a systemd unit configuration file to handle process initialization, monitoring, and security containment rules.

```bash
sudo nano /etc/systemd/system/node-app.service
```

Paste the following production-hardened configuration:

```ini
[Unit]
Description=Production Node.js Application
After=network.target

[Service]
Type=simple
User=nodeuser
Group=nodeuser
WorkingDirectory=/var/www/my-node-app
ExecStart=/usr/bin/node server.js
Restart=on-failure

# Hardening / Security measures
ProtectSystem=full
ProtectHome=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

---

## 🚀 Step 5: Start and Enable the Service

Reload the system daemon to load your new service definitions, start the Node.js application process, and ensure it boots up automatically on system reboot.

```bash
# 1. Reload systemd configurations
sudo systemctl daemon-reload

# 2. Start the service
sudo systemctl start node-app

# 3. Enable auto-start at system boot
sudo systemctl enable node-app
```

---

## 📊 Step 6: Verification and Auditing

Verify that everything is running perfectly and securely inside the background.

```bash
# 1. Check system service status logs
sudo systemctl status node-app

# 2. Verify the process is running strictly under 'nodeuser'
ps -ef | grep node

# 3. View live application logging outputs
sudo journalctl -u node-app.service -f
```
