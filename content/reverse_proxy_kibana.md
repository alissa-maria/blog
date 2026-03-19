---
title: "Setting up Kibana behind a reverse proxy (Nginx)"
description: "For my first guide, I'd like to explain in an easy-to-follow manner how to set up Kibana to work behind a reverse proxy, specifically Nginx."
date: 2025-01-24
categories: ["Web"]
tags: ["Kibana", "Nginx"]
---

This guide walks you through setting up Kibana behind an Nginx reverse proxy, including enabling SSL with Let's Encrypt, configuring DNS, and optionally securing access with HTTP Basic Authentication, on **Ubuntu 22.04 LTS**.

When you’re new to Elasticsearch and Kibana, the configuration process can feel overwhelming, like one big maze. I’ve tried to make this tutorial as easy to follow as possible, breaking it into straightforward steps. Let’s dive in! 🦦

## Step 1: Install Nginx

First, ensure your server is up-to-date and install Nginx.

```bash
sudo apt-get update
sudo apt-get install nginx
```

## Step 2: Configure DNS Records

Point your domain name (e.g., `kibana.mydomain.net`) to the server's public IP address using DNS records. This ensures your domain resolves to the correct server.

## Step 3: Update Kibana Configuration

Modify the Kibana configuration file (`/etc/kibana/kibana.yml`) to prepare it for use with the reverse proxy.

### 1. **Bind Kibana to `localhost`**
Ensure Kibana only listens on `localhost` so it is not directly accessible to the public:

```yaml
server.host: "127.0.0.1"
```

### 2. **Set the public-facing URL**
Specify the `server.publicBaseUrl` to inform Kibana of its public URL. This is essential for generating proper links in emails, logs, and notifications:

```yaml
server.publicBaseUrl: "https://kibana.mydomain.net"
```

### 3. **Enable proxy headers**
Ensure Kibana correctly interprets headers from the reverse proxy. This is crucial for maintaining the correct IP addresses and SSL information. Add the following:

```yaml
server.rewriteBasePath: false
```

Set the `xpack.security.secureCookies` field to `true` if you are using HTTPS:

```yaml
xpack.security.secureCookies: true
```

### 4. **Disable anonymous access (optional)**
If you’ve enabled X-Pack security features, ensure that anonymous access is disabled. This will prevent unauthenticated users from bypassing the proxy:

```yaml
xpack.security.authc.anonymous.enabled: false
```

### Restart Kibana
After making these changes, restart Kibana to apply the updated configuration:

```bash
sudo systemctl restart kibana
```

## Step 4: Configure the Nginx site

### Create an Nginx server block

Create a new configuration file for Kibana in `/etc/nginx/sites-available/kibana`:

```nginx
server {
    listen 80;
    server_name kibana.mydomain.net;

    location / {
        proxy_pass http://127.0.0.1:5601;  # Forward requests to Kibana
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

You’ll notice there’s no SSL configuration here yet. This will be automatically added when we set up a Let's Encrypt certificate in Step 6.

### Enable the configuration and test

Create a symbolic link to enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
```

Then test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```

If the test passes, restart Nginx:

```bash
sudo service nginx restart
```

## Step 5: Set up SSL with Let's Encrypt (Certbot)

Install Certbot and obtain an SSL certificate for your domain:

```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d kibana.mydomain.net
```

Verify the output. If your DNS record is correctly configured, there should be no issues. Certbot will automatically configure SSL settings for you, including redirection from HTTP to HTTPS.

## Summary

With this setup, you've successfully configured Kibana behind an Nginx reverse proxy with SSL. Your application is now secure and accessible at https://kibana.mydomain.net. If you’d like to add additional protection, such as IP whitelisting, I recommend referring to the official Nginx documentation for detailed guidance.

Feel free to extend this setup for additional domains or services as needed! 💕
