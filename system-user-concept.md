# Why Software Runs Under Non-Human (System) Users

We run applications and software with non-human (system) users primarily for **security isolation, automation, and operational continuity**. 

If an application runs under your personal human user account, it inherits all of your personal permissions. If that application gets compromised by an attacker, the attacker instantly gains access to your private files, your SSH keys, and potentially your administrative privileges.

---

## 🛡️ Core Reasons for Non-Human Users

### 1. The Principle of Least Privilege (Security Isolation)
Non-human users are intentionally stripped of almost all privileges. 
* **Blast Radius Containment:** If a hacker exploits a vulnerability in a Node.js web application running as a non-human `nodeuser`, they are trapped inside that user's sandbox. They cannot read your human user's home directory, access personal documents, or run `sudo` commands to take over the machine.
* **No Login Capabilities:** Non-human users usually have their login shell set to `/bin/false` or `/sbin/nologin`. Even if a hacker steals the password hash for that user, they cannot use it to log into your server via SSH.

### 2. Automation and Server Independence
Human users log in, perform tasks, and log out. Applications need to run permanently.
* **Continuous Uptime:** If you launch a web server using your human account, that process is tied to your login session. When you log out or disconnect your SSH terminal, the operating system will terminate your processes.
* **Unattended Booting:** When a server reboots at 3:00 AM due to an update, non-human users allow background services (like MySQL or Nginx) to start up immediately without waiting for a real human to type in a username and password.

### 3. Traceability and Auditing
In a production environment, keeping human actions separate from system actions makes troubleshooting and security audits much easier.
* If a file is modified by `john_doe` (human), you know an administrator did it manually.
* If a file is modified by `www-data` (non-human web server user), you know the web application itself generated that file automatically.

---

## 📊 Quick Comparison: Human vs. Non-Human Users

| Feature | Human User (e.g., `ubuntu`, `john`) | Non-Human User (e.g., `nodeuser`, `mysql`) |
| :--- | :--- | :--- |
| **Interactive Login** | Yes (Can use SSH, Terminal, GUI) | **No** (Blocked by `/bin/false`) |
| **Permissions** | Broad (Can access personal files, run `sudo`) | **Strictly Limited** (Only accesses its specific app folder) |
| **Lifecycle** | Tied to active login sessions | **Persistent** (Runs in background via system services) |
