---
title: "Setting up Kibana behind a reverse proxy (Nginx)"
description: "A straightforward guide to placing Kibana behind an Nginx reverse proxy."
created: 2025-01-24
# categories: ["Web"]
tags:
  - kibana
  - nginx
---

This guide walks through setting up Kibana behind an Nginx reverse proxy on **Ubuntu 22.04 LTS**, including DNS configuration, SSL with Let's Encrypt, and optional access restrictions.

It assumes that Elasticsearch and Kibana are already installed and running, and focuses on configuring the reverse proxy layer with Nginx.

When you're new to Elasticsearch and Kibana, the configuration can feel a bit maze-like. I wrote this because I couldn't find a single guide that covered the full setup end-to-end. It often requires a mix of skills (DNS, web servers, certificates), which makes it harder than it should be if you're just trying to get Kibana working.

The steps below are kept straightforward and easy to verify as you go.

## ✧ Step 1: Install Nginx

First, update the server and install Nginx.

```bash
sudo apt-get update
sudo apt-get install nginx
```

## ✧ Step 2: Configure DNS records

Point your domain name (for example, `kibana.mydomain.net`) to the server's public IP address. This ensures the domain resolves to the correct host.

## ✧ Step 3: Update the Kibana configuration

Modify `/etc/kibana/kibana.yml` so Kibana is ready to sit behind the reverse proxy.

**1. Bind Kibana to `localhost`**

Ensure Kibana only listens on `localhost`, so it is not directly exposed to the public internet.

```yaml
server.host: "127.0.0.1"
```

**2. Set the public-facing URL**

Set `server.publicBaseUrl` so Kibana knows its external URL. This matters for generated links in emails, logs, and notifications.

```yaml
server.publicBaseUrl: "https://kibana.mydomain.net"
```

**3. Enable proxy-related settings**

Keep `server.rewriteBasePath` set to `false` unless you are serving Kibana from a subpath rather than the domain root.

```yaml
server.rewriteBasePath: false
```

If you are serving Kibana over HTTPS, also set:

```yaml
xpack.security.secureCookies: true
```

**4. Disable anonymous access (optional)**

If X-Pack security is enabled, make sure anonymous access is disabled so unauthenticated users cannot bypass the proxy layer.

```yaml
xpack.security.authc.anonymous.enabled: false
```

**Restart Kibana**

After making these changes, restart Kibana:

```bash
sudo systemctl restart kibana
```

## ✧ Step 4: Configure the Nginx site

**Create an Nginx server block**

Create `/etc/nginx/sites-available/kibana`:

```nginx
server {
    listen 80;
    server_name kibana.mydomain.net;

    location / {
        proxy_pass http://127.0.0.1:5601;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

There is no SSL configuration here yet. Certbot will add that later.

**Enable the configuration and test it**

Create a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
```

Then test the configuration:

```bash
sudo nginx -t
```

If the test passes, restart Nginx:

```bash
sudo service nginx restart
```

> [!tip]
>
> If everything looks correct but it still doesn’t work, it’s often DNS. It’s worth checking your provider’s portal to confirm the records and TTL.

## ✧ Step 5: Set up SSL with Let's Encrypt

Install Certbot and request a certificate for the domain:

```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d kibana.mydomain.net
```

If the DNS record is correct, Certbot should be able to issue the certificate and update the Nginx configuration automatically, including HTTP-to-HTTPS redirection.

## ✧ Summary

At this point, Kibana should be reachable securely at `https://kibana.mydomain.net` through Nginx.

If you want to lock it down further, for example with IP allowlisting or additional authentication layers, the official Nginx documentation is a good next stop.