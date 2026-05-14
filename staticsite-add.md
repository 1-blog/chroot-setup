```bash
nano /etc/nginx/sites-available/staticsite
```

```bash
server {
    listen 80;
    listen [::]:80;

    server_name static.balibabu.duckdns.org;

    root /var/www/staticsite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/staticsite /etc/nginx/sites-enabled/
```

```bash
nginx -t
```

```bash
nginx -s reload
```
```bash
certbot --nginx -d static.balibabu.duckdns.org
```
