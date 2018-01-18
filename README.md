Install a reverse-proxy using Nginx and Let's Encrypt SSL (certbot) to access Odoo through HTTPS. This was tested with Odoo 11 on Ubuntu 16.04 VPS.

## 1. Update your server
```
sudo apt-get update -y && sudo apt-get dist-upgrade -y && apt-get auto-remove -y
```
## 2. Install Odoo
If not done yet, install Odoo with [Yenthe666's script].
## 3. Install Nginx
``` 
sudo apt-get install nginx -y
```
## 4. Certbot
#### Install Certbot
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```
### Run Certbot
```
sudo nginx stop
```
```
sudo certbot --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
```
### Automating renewal


### Installation procedure
```
sudo nano /etc/nginx/sites-available/YOURDOMAIN.conf
```
Paste the code bellow and adapt it.
```
upstream odoo {
    server 127.0.0.1:8069;
}

server {
    listen      443 default;
    server_name www.YOURDOMAIN.COM;
    
    access_log  /var/log/nginx/oddo.access.log;
    error_log   /var/log/nginx/oddo.error.log;

    ssl on;
    ssl_certificate     /etc/letsencrypt/live/www.YOURDOMAIN.COM/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.YOURDOMAIN.COM/privkey.pem;
    keepalive_timeout   60;

    ssl_protocols TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-$
    ssl_prefer_server_ciphers on;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass  http://odoo;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto https;
    }

    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo;
    }
}

server {
    listen      80;
    server_name www.YOURDOMAIN.COM;

    add_header Strict-Transport-Security max-age=2592000;
    rewrite ^/.*$ https://$host$request_uri? permanent;
}
```
Create a symlink
```
sudo ln -s /etc/nginx/sites-available/YOURDOMAIN.conf /etc/nginx/sites-enabled/YOURDOMAIN.conf
```
Reload Nginx
```
sudo nginx reload
```
Your Odoo instance should now be accessible through HTTPS.
