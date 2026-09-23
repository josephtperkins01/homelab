# Nginx Virtual Hosts

I configured Nginx to serve two independent sites from the same Ubuntu VM. This gave me a practical example of server blocks, host-based routing, and troubleshooting a default-site conflict without needing public DNS.

## Sites I built

| Site | Document root | Listener |
|------|---------------|----------|
| `site1.local` | `/var/www/site1.local/html` | Port 80 |
| `site2.local` | `/var/www/site2.local/html` | Port 8081 |

I used port 8081 for Site 2 so it could coexist with the existing port-80 setup during testing.

## Create the content

```bash
sudo mkdir -p /var/www/site1.local/html
sudo mkdir -p /var/www/site2.local/html

sudo nano /var/www/site1.local/html/index.html
sudo nano /var/www/site2.local/html/index.html
```

Each page has different text, which made it obvious when Nginx routed a request to the wrong site.

## Server blocks

I saved these files in `/etc/nginx/sites-available/`.

`site1.local`:

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

`site2.local`:

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

I enabled both configurations and validated before reloading:

```bash
sudo ln -s /etc/nginx/sites-available/site1.local /etc/nginx/sites-enabled/site1.local
sudo ln -s /etc/nginx/sites-available/site2.local /etc/nginx/sites-enabled/site2.local
sudo nginx -t
sudo systemctl reload nginx
```

## The default-server problem

My first test of `site1.local` returned the original Nginx page instead of Site 1's page. The problem was not the document root. Several server blocks were listening on port 80, and none explicitly declared `default_server`, so Nginx selected the first loaded configuration when the request did not match a `server_name`.

Because Site 1 replaced the original default page in this lab, I removed the old enabled site and tested again:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

In a larger environment, I would use an explicit fallback `default_server` instead of removing the distribution default.

## How I verified routing without DNS

I used custom `Host` headers to simulate what a DNS-resolved browser request would send:

```bash
curl -H "Host: site1.local" http://localhost/
curl -H "Host: site2.local" http://localhost:8081/
```

The first command returned Site 1 content and the second returned Site 2 content. This confirmed that the server blocks, document roots, and listener ports were all working together.

For browser testing, I can add the names to a local hosts file:

```text
127.0.0.1 site1.local
127.0.0.1 site2.local
```

Last updated: September 23, 2026
