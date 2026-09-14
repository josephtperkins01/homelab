# Nginx Virtual Hosts

This document records the multi-site Nginx configuration used in the homelab.
Two independent simulated sites are hosted on the same Ubuntu server without
requiring public DNS.

## Sites

| Site | Document root | Listener |
|------|---------------|----------|
| `site1.local` | `/var/www/site1.local/html` | Port 80 |
| `site2.local` | `/var/www/site2.local/html` | Port 8081 |

`site2.local` uses port 8081 to avoid conflicting with the existing default
site while the second site was being introduced.

## Create site directories and pages

```bash
sudo mkdir -p /var/www/site1.local/html
sudo mkdir -p /var/www/site2.local/html

sudo nano /var/www/site1.local/html/index.html
sudo nano /var/www/site2.local/html/index.html
```

Each page contains site-specific content so routing can be verified.

## Create server blocks

Create `/etc/nginx/sites-available/site1.local`:

```nginx
server {
    listen 80;
    server_name site1.local;

    root /var/www/site1.local/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Create `/etc/nginx/sites-available/site2.local`:

```nginx
server {
    listen 8081;
    server_name site2.local;

    root /var/www/site2.local/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## Enable the sites

```bash
sudo ln -s /etc/nginx/sites-available/site1.local \
  /etc/nginx/sites-enabled/site1.local
sudo ln -s /etc/nginx/sites-available/site2.local \
  /etc/nginx/sites-enabled/site2.local

sudo nginx -t
sudo systemctl reload nginx
```

## Default-site troubleshooting

During initial testing, requests to `site1.local` returned the old default
site. Multiple server blocks were listening on port 80, and no explicit
`default_server` directive was present. Nginx therefore selected the first
loaded server block as the default for requests that did not match a
`server_name`.

Because `site1.local` superseded the original default page, the old default
site was removed:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

In a larger deployment, an explicit `default_server` or a dedicated fallback
server block may be preferable to removing the distribution default.

## Verify host-based routing

No DNS entry is required for local testing. The `Host` header simulates the
hostname that a browser would send after DNS resolution:

```bash
curl -H "Host: site1.local" http://localhost/
curl -H "Host: site2.local" http://localhost:8081/
```

Expected results:

- The first command returns Site 1 content from `/var/www/site1.local/html`.
- The second command returns Site 2 content from `/var/www/site2.local/html`.

## Optional local name resolution

For browser testing, add the names to the Ubuntu or Windows hosts file rather
than exposing public DNS:

```text
127.0.0.1 site1.local
127.0.0.1 site2.local
```

When using the VirtualBox HTTP port forward, map the browser's host port to
the appropriate guest port. Host-header testing with `curl` remains the most
direct way to verify Nginx routing.

## Firewall note

If port 8081 is accessed through a VirtualBox forward, allow it explicitly in
UFW and create the corresponding forward:

```bash
sudo ufw allow 8081/tcp
sudo ufw status
```

## Lessons demonstrated

- Nginx server blocks isolate sites on one server.
- `server_name` controls host-based routing.
- Listener ports can separate sites during local testing.
- `nginx -t` should run before every reload.
- Default-server selection matters when multiple blocks share a port.

Last updated: September 14, 2026
