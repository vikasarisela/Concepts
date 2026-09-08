Since you're learning RHEL troubleshooting, use this checklist in this order.

Nginx troubleshooting checklist — RHEL
1. Check whether Nginx is installed
rpm -q nginx

If not installed:

dnf install nginx -y
2. Check the service status
systemctl status nginx

Look for:

Active: active (running)

If stopped:

systemctl start nginx
3. Check whether Nginx starts automatically
systemctl is-enabled nginx

If you want it to start after reboot:

systemctl enable nginx
4. Check Nginx configuration

Very important:

nginx -t

Expected:

syntax is ok
test is successful

If this fails, fix the configuration before restarting Nginx.

5. Check whether Nginx is listening on port 80
ss -lntp | grep :80

Expected:

0.0.0.0:80

or:

[::]:80
6. Test Nginx locally
curl http://localhost

If you get HTML, Nginx is responding.

You can also test:

curl http://127.0.0.1
7. Check the Linux firewall
firewall-cmd --list-all

Look for:

services: ... http ...

If http is missing:

firewall-cmd --permanent --add-service=http
firewall-cmd --reload

Then verify:

firewall-cmd --list-services
8. Check Nginx logs

Error log:

tail -f /var/log/nginx/error.log

Access log:

tail -f /var/log/nginx/access.log

Also useful:

journalctl -u nginx

For recent errors:

journalctl -u nginx -xe
9. Check the Nginx processes
ps aux | grep nginx

You should normally see a master process and one or more worker processes.

nginx: master process
nginx: worker process
10. Check the web page/document root

For the default Nginx configuration, check:

ls -l /usr/share/nginx/html/

Try:

curl http://localhost

If you get 403 Forbidden, investigate permissions, SELinux, or the Nginx configuration.

11. Check SELinux

Check status:

getenforce

If it says:

Enforcing

SELinux may be involved if you've changed document roots, ports, or file contexts.

Check recent SELinux denials:

ausearch -m AVC -ts recent
12. Check which port Nginx is configured to use
grep -R "listen" /etc/nginx/

You might see:

listen 80;

If Nginx is configured for another port, your browser must use that port.

13. Restart after configuration changes
nginx -t
systemctl restart nginx

Always run nginx -t before restarting after configuration changes.

🧠 Your troubleshooting flow

Memorize this:

1. Is nginx installed?
       ↓
2. Is nginx service running?
       ↓
3. Is configuration correct? (nginx -t)
       ↓
4. Is Nginx listening? (ss -lntp)
       ↓
5. Does localhost work? (curl)
       ↓
6. Is firewall allowing HTTP?
       ↓
7. Check logs
       ↓
8. Check SELinux
       ↓
9. Check configuration/document root
⭐ The 5 commands I'd memorize first
systemctl status nginx
nginx -t
ss -lntp | grep :80
curl http://localhost
firewall-cmd --list-all

These five will solve or quickly narrow down most basic Nginx problems on RHEL.