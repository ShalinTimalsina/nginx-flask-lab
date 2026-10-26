# Hosting a Static Website

This lab shows you how to serve a simple HTML page using Nginx.

---

## Files

- `index.html`: The web page that will be shown to visitors.
- `mysite.conf`: The Nginx virtual host configuration file.

---

## Step-by-Step Instructions

### 1. Copy website files
```bash
sudo mkdir -p /var/www/mysite/html
sudo cp index.html /var/www/mysite/html/
```

### 2. Copy the Nginx config
```bash
sudo cp mysite.conf /etc/nginx/sites-available/mysite
```

### 3. Turn the site ON (create a symlink)
```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

### 4. Test config and reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 5. Add domain to your machine's hosts file
```bash
echo '127.0.0.1  mysite.local' | sudo tee -a /etc/hosts
```

### 6. View the site
```bash
curl http://mysite.local
```
Or open `http://mysite.local` in your browser.
