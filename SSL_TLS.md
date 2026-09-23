# SSL/TLS Configuration - Self-Signed Certificate

I used a self-signed certificate because this VM has no public DNS name or reachable public IP. That let me practice the Nginx TLS configuration without exposing the lab.


This document records the SSL/TLS setup using a self-signed certificate for the homelab Nginx server.

## Overview

A self-signed SSL/TLS certificate allows you to:
- Serve content over HTTPS (encrypted)
- Demonstrate production-level SSL/TLS configuration skills
- Work in a lab environment without a public domain or Let's Encrypt

**Important:** Self-signed certificates trigger browser security warnings (expected behavior). Browsers warn because the cert isn't from a trusted Certificate Authority (CA), not because anything is wrong with the setup.

## Why Self-Signed in a Home Lab?

Let's Encrypt requires:
- A real, publicly-registered domain name
- A publicly reachable IP address
- The ability to respond to ACME challenges

A home lab behind VirtualBox NAT doesn't meet these requirements without exposing the server to the internet (not recommended).

Self-signed certs are perfect for:
- Learning SSL/TLS configuration
- Demonstrating production Nginx configs locally
- Lab/portfolio projects (with context explained)

## Certificate Generation

### Step 1: Create SSL directory

```bash
sudo mkdir -p /etc/nginx/ssl
```

### Step 2: Generate self-signed certificate and key

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
```

**Prompts (you can press Enter to skip or enter values):**
- Country Name (2 letter code)
- State or Province Name
- Locality Name (city)
- Organization Name
- Organizational Unit Name
- Common Name (e.g., homelab-server or localhost) — **recommended to enter this one**
- Email Address

**Result:**
- Private key: `/etc/nginx/ssl/selfsigned.key` (keep secure)
- Certificate: `/etc/nginx/ssl/selfsigned.crt` (public, safe to share)

### Verify certificate was created

```bash
ls -la /etc/nginx/ssl/
# Expected: both .key and .crt files present

# View certificate details
sudo openssl x509 -in /etc/nginx/ssl/selfsigned.crt -text -noout
```

## Nginx Configuration

### Edit Nginx config

```bash
sudo nano /etc/nginx/sites-available/default
```

### Add HTTPS server block

Add this block **before** the existing HTTP server block:

```nginx
server {
    listen 443 ssl http2;
    server_name homelab-server;

    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    # SSL best practices (optional but recommended)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    root /var/www/html;
    index index.html index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Optional: Redirect HTTP to HTTPS

To force all traffic to use HTTPS, add this to the existing HTTP server block (port 80):

```nginx
server {
    listen 80;
    server_name homelab-server;

    # Redirect all HTTP traffic to HTTPS
    return 301 https://$server_name$request_uri;
}
```

### Test configuration

```bash
sudo nginx -t
# Expected: "nginx: configuration file /etc/nginx/nginx.conf syntax is ok"
```

### Reload Nginx

```bash
sudo systemctl reload nginx
# or
sudo systemctl restart nginx
```

## Firewall Rules

Allow HTTPS traffic:

```bash
# Allow port 443 (HTTPS)
sudo ufw allow 443/tcp

# Verify
sudo ufw status
# Expected: 443/tcp ALLOW IN Anywhere
```

## VirtualBox Port Forwarding

Add a new port-forwarding rule:

| Setting | Value |
|---------|-------|
| Name | HTTPS |
| Protocol | TCP |
| Host IP | 127.0.0.1 |
| Host Port | 8443 |
| Guest IP | (leave blank) |
| Guest Port | 443 |

## Access from Windows

### Using browser

1. Open browser on Windows
2. Navigate to: `https://127.0.0.1:8443`
3. Browser will warn: "Your connection is not private" (or similar)
4. Click "Advanced" → "Proceed to 127.0.0.1 (unsafe)" to continue
5. Page loads successfully over HTTPS

### Using PowerShell (ignore self-signed warning)

```powershell
# Note: curl will warn about self-signed cert, but connection works
curl --insecure https://127.0.0.1:8443

# Or use Invoke-WebRequest
Invoke-WebRequest -Uri https://127.0.0.1:8443 -SkipCertificateCheck
```

## Certificate Details

### Check certificate expiration

```bash
sudo openssl x509 -in /etc/nginx/ssl/selfsigned.crt -noout -dates
# Output: notBefore=... notAfter=...
```

The certificate was generated with `-days 365`, so it expires 1 year from creation date.

### Renew certificate (before expiration)

Simply regenerate the certificate with the same command:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
sudo systemctl reload nginx
```

## Browser Security Warning Explained

When visiting `https://127.0.0.1:8443`, you'll see a warning like:

```
Your connection is not private

Attackers might be trying to steal your information from 127.0.0.1
(for example, passwords, messages, or credit cards).

NET::ERR_CERT_AUTHORITY_INVALID
```

**What this means:**
- ✅ The connection IS encrypted (HTTPS works)
- ✅ The certificate IS valid for `127.0.0.1`
- ❌ The certificate isn't from a trusted Certificate Authority (CA)

**Why it's expected:**
- Your cert was self-signed (not signed by a trusted CA like Let's Encrypt)
- Browsers trust only certificates from recognized CAs by default
- This is a **lab environment security feature, not a configuration error**

**In production:**
- Use a certificate from a trusted CA (Let's Encrypt, DigiCert, etc.)
- Browsers will trust it automatically
- No warnings

## Advanced: HTTP/2 with SSL

The config above includes `http2` for better performance over HTTPS:

```nginx
listen 443 ssl http2;
```

This enables HTTP/2, which multiplexes requests for faster page loads.

## Troubleshooting

### Nginx won't start after SSL config

```bash
# Check for syntax errors
sudo nginx -t

# View error log
sudo tail -20 /var/log/nginx/error.log

# Common issues:
# - Wrong path to cert/key file
# - Missing/incorrect ssl_certificate directives
# - Port already in use
```

### Certificate path not found

Verify paths match exactly:

```bash
# Check files exist
ls -la /etc/nginx/ssl/
sudo cat /etc/nginx/ssl/selfsigned.crt | head
sudo cat /etc/nginx/ssl/selfsigned.key | head
```

### Can't access HTTPS from Windows

1. Verify Nginx is running: `sudo systemctl status nginx`
2. Verify port 443 is listening: `sudo ss -tlnp | grep 443`
3. Verify firewall allows 443: `sudo ufw status`
4. Verify VirtualBox port-forward rule exists (Host 8443 → Guest 443)
5. Test from VM: `curl -k https://127.0.0.1`

### Browser still warns after adding cert to trusted store

Self-signed certs added to a single browser don't affect other browsers or applications. Each tool must be configured separately to trust the cert (or you can ignore the warning, which is standard for lab environments).

## Migration to Let's Encrypt (future, if you get a public domain)

If you eventually get a public domain pointing to your server:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot certonly --nginx -d yourdomain.com
# Certbot will automatically configure Nginx with the Let's Encrypt cert
```

Then the browser warnings disappear and you have a production-trusted certificate.

---

**Last Updated:** August 28, 2026
**Certificate Type:** Self-signed, X.509
**Algorithm:** RSA 2048-bit
**Valid For:** 365 days
**Author:** josephtperkins01
