---
title: "nginx configuration for exposing harbor"
date: 2026-01-15T11:25:00-06:00
draft: false
tags:
  - nginx
  - harbor
---

## nginx configuration for harbor

* Create the DNS records in your public DNS
* Generate the certificates for your external harbor endpoint. I use letsencrypt.
* Setup [harbor](https://github.com/goharbor/harbor) on your virtual machine. Ensure you assign a decent amount of storage to this vm. Ensure you set this up to listen on HTTP, not HTTPS
* Use the following nginx config to do the following
  - Terminate SSL
  - Proxy to the internal harbor vm
  - Ensure the large images are supported

```
server {
    listen                  443 ssl;
    listen                  [::]:443 ssl;
    http2                   on;
    server_name             harbor.public.url;

    ssl_protocols TLSv1.2 TLSv1.3;

    client_max_body_size 0;

    proxy_read_timeout 3600;
    proxy_send_timeout 3600;

    # SSL
    ssl_certificate /etc/letsencrypt/live/public.url/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/public.url/privkey.pem; # managed by Certbot
    ssl_trusted_certificate /etc/letsencrypt/live/public.url/chain.pem;
    # security
    #include                 nginxconfig/security.conf;

    # REQUIRED HEADERS
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Authorization     $http_authorization;

    proxy_http_version 1.1;
    proxy_set_header Connection "";

    proxy_redirect off;
    proxy_buffering off;

    # logging
    access_log /var/log/nginx/access.log combined buffer=512k flush=1m;
    error_log  /var/log/nginx/error.log warn;

    location / {
        proxy_pass "http://harbor.internal.url/";
    }

    location /service/ {
        proxy_pass "http://harbor.internal.url/service/";
    }

    location /v2/ {
        proxy_pass "http://harbor.internal.url/v2/";
    }
}

server {
    if ($host ~ ^[^.]+\.public\.url$) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen      80;
    listen      [::]:80;
    server_name .public.url;
    return 404; # managed by Certbot
}
```

**NOTE: Change the `.public.url` and `.internal.url` to map to your external and internal domains**

* Upload this file into your nginx server and place it in `/etc/nginx/sites-available/harbor.conf`
* Next link this file `sudo ln -s /etc/nginx/sites-available/harbor.conf /etc/nginx/sites-enabled`
* Validate the config file `sudo nginx -t`
* If all is well, then `sudo systemctl reload nginx`

Now hopefully you can use `docker` or `podman` to pull and push images over to your harbor

**Side note: Don't expose any of your projects on public without any authentication**

Have fun building images.