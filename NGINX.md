# Nginx Configuration

This document records the Nginx web server installation and configuration applied to the homelab.

## Overview

Nginx is a lightweight, high-performance web server. It's installed and configured to serve a custom homepage and can be extended for virtual hosts, reverse proxying, and SSL/TLS.

![Nginx Homepage Screenshot](../screenshots/nginx-homepage.png)

*Nginx serving custom homepage on 127.0.0.1:8080 with security hardening applied*


## Installation

```bash
sudo apt update
sudo apt install -y nginx
```

## Service Management

```bash
# Check status
sudo systemctl status nginx

# Start/stop/restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Enable on boot
sudo systemctl enable nginx
```

## Configuration

Nginx configuration files are located in `/etc/nginx/`:

- `/etc/nginx/nginx.conf` — Main configuration
- `/etc/nginx/sites-available/` — Available site configurations
- `/etc/nginx/sites-enabled/` — Enabled site configurations (symlinks)
- `/etc/nginx/conf.d/` — Additional configuration snippets

## Default Site

The default site is configured to serve content from `/var/www/html/`.

### Create custom homepage

```bash
sudo nano /var/www/html/index.html
```

Example content:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to my homelab server</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        h1 { color: #333; }
        p { color: #666; }
    </style>
</head>
<body>
    <h1>Welcome to my homelab server</h1>
    <p>Nginx is running and serving this page.</p>
    <p>Hostname: <strong>homelab-server</strong></p>
    <p>Status: Active and hardened</p>
</body>
</html>
```

After editing:
```bash
# Verify syntax
sudo nginx -t
# Expected: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

# Reload configuration
sudo systemctl reload nginx
```

## Firewall Rules

To access Nginx from Windows, add firewall rules:

```bash
# Allow HTTP (port 80)
sudo ufw allow 80/tcp

# Allow HTTPS (port 443) — for future SSL setup
sudo ufw allow 443/tcp

# Verify
sudo ufw status
```

## Port Forwarding

Add VirtualBox port-forwarding rules to expose Nginx:

**HTTP:**
- Name: HTTP
- Protocol: TCP
- Host IP: 127.0.0.1
- Host Port: 8080
- Guest IP: (leave blank)
- Guest Port: 80

**HTTPS (optional, for future):**
- Name: HTTPS
- Protocol: TCP
- Host IP: 127.0.0.1
- Host Port: 8443
- Guest IP: (leave blank)
- Guest Port: 443

## Access from Windows

Once port-forwarding is configured:

```powershell
# Open browser and navigate to:
http://127.0.0.1:8080

# Or from PowerShell:
curl http://127.0.0.1:8080
```

## Verification

From the VM:

```bash
# Check Nginx is listening
sudo ss -tlnp | grep nginx
# or
sudo netstat -tlnp | grep nginx

# Expected output shows nginx listening on port 80 (and 443 if configured)
```

## Virtual Hosts (optional)

To host multiple sites:

1. Create site configuration in `/etc/nginx/sites-available/`:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Example:
```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

2. Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

3. Verify and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

4. (Optional) Create the site directory and content:

```bash
sudo mkdir -p /var/www/example.com
sudo nano /var/www/example.com/index.html
```

## SSL/TLS Setup (future)

To add HTTPS support using Let's Encrypt Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx

# Request a certificate (requires a real domain)
sudo certbot certonly --nginx -d example.com

# Nginx will automatically use the certificate (Certbot configures it)

# Verify auto-renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

## Logs

- **Access log:** `/var/log/nginx/access.log`
- **Error log:** `/var/log/nginx/error.log`

View logs:

```bash
# Tail access log
sudo tail -f /var/log/nginx/access.log

# Tail error log
sudo tail -f /var/log/nginx/error.log

# Check for errors after configuration changes
sudo tail -20 /var/log/nginx/error.log
```

## Troubleshooting

### Nginx won't start

```bash
# Check for syntax errors
sudo nginx -t

# Check logs
sudo tail -20 /var/log/nginx/error.log

# Check if port 80 is in use
sudo ss -tlnp | grep :80
```

### Can't access from Windows

1. Verify Nginx is running: `sudo systemctl status nginx`
2. Verify firewall allows port 80: `sudo ufw status`
3. Verify port-forwarding rule in VirtualBox (8080 → 80)
4. Test from VM: `curl http://127.0.0.1`
5. Test from Windows: `curl http://127.0.0.1:8080`

### Configuration reload fails

```bash
# Always verify before reloading
sudo nginx -t

# If test fails, fix the error and try again
# Restart if reload doesn't work
sudo systemctl restart nginx
```

---

**Last Updated:** August 28, 2026
**Version:** Nginx (latest stable Ubuntu package)
**Author:** josephtperkins01
