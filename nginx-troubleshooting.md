# Nginx Troubleshooting Checklist — RHEL

## 1. Check if Nginx is installed

```bash
rpm -q nginx
```

If Nginx is not installed:

```bash
dnf install nginx -y
```

---

## 2. Check Nginx service status

```bash
systemctl status nginx
```

Expected:

```text
Active: active (running)
```

If Nginx is stopped:

```bash
systemctl start nginx
```

---

## 3. Check if Nginx starts automatically after reboot

```bash
systemctl is-enabled nginx
```

Enable it if required:

```bash
systemctl enable nginx
```

---

## 4. Check Nginx configuration

Before restarting Nginx, always test the configuration:

```bash
nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

If the test fails, fix the configuration error first.

---

## 5. Check if Nginx is listening on port 80

```bash
ss -lntp | grep :80
```

Expected:

```text
0.0.0.0:80
[::]:80
```

This means Nginx is listening for HTTP connections on port 80.

---

## 6. Test Nginx locally

```bash
curl http://localhost
```

You can also test:

```bash
curl http://127.0.0.1
```

If HTML is returned, Nginx is responding locally.

---

## 7. Check the Linux firewall

```bash
firewall-cmd --list-all
```

Look for:

```text
services: ... http ...
```

If HTTP is not allowed:

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

Verify:

```bash
firewall-cmd --list-services
```

---

## 8. Check Nginx processes

```bash
ps aux | grep nginx
```

Normally you should see:

```text
nginx: master process
nginx: worker process
```

Check the main PID:

```bash
systemctl status nginx
```

Example:

```text
Main PID: 7199 (nginx)
```

---

## 9. Check Nginx logs

### Error log

```bash
tail -f /var/log/nginx/error.log
```

### Access log

```bash
tail -f /var/log/nginx/access.log
```

### Systemd logs

```bash
journalctl -u nginx
```

For detailed recent errors:

```bash
journalctl -u nginx -xe
```

---

## 10. Check Nginx configuration for listening port

```bash
grep -R "listen" /etc/nginx/
```

Example:

```text
listen 80;
```

If Nginx is listening on another port, use that port when accessing it.

---

## 11. Check the web root

For the default Nginx configuration:

```bash
ls -l /usr/share/nginx/html/
```

Test the default page:

```bash
curl http://localhost
```

---

## 12. Check SELinux

Check SELinux status:

```bash
getenforce
```

Possible output:

```text
Enforcing
```

If you suspect SELinux is blocking Nginx, check for denials:

```bash
ausearch -m AVC -ts recent
```

---

## 13. Restart Nginx after configuration changes

First test the configuration:

```bash
nginx -t
```

If successful:

```bash
systemctl restart nginx
```

Then verify:

```bash
systemctl status nginx
```

---

# Quick Troubleshooting Flow

```text
Is Nginx installed?
        ↓
rpm -q nginx
        ↓
Is the service running?
        ↓
systemctl status nginx
        ↓
Is the configuration correct?
        ↓
nginx -t
        ↓
Is Nginx listening on port 80?
        ↓
ss -lntp | grep :80
        ↓
Does localhost work?
        ↓
curl http://localhost
        ↓
Is HTTP allowed by firewall?
        ↓
firewall-cmd --list-all
        ↓
Check logs
        ↓
journalctl -u nginx
        ↓
Check SELinux if necessary
```

# ⭐ 5 Commands to Remember

```bash
systemctl status nginx
nginx -t
ss -lntp | grep :80
curl http://localhost
firewall-cmd --list-all
```

## 🧠 Simple Memory

**Installed → Running → Config → Listening → Local Test → Firewall → Logs → SELinux**