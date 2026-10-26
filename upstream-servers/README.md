# Module 10: Upstream Servers (Ports 3000 and 3001)

This lab shows you how to route traffic to two different apps from one single port 80 domain:

```
                      ┌──> /app1/ ──> App One (Port 3000)
Browser ──> Nginx :80 │
                      └──> /app2/ ──> App Two (Port 3001)
```

---

## Files

- `app1/index.html`: Web page for App One (`This is App One`).
- `app2/index.html`: Web page for App Two (`This is App Two`).
- `myapps.conf`: The Nginx configuration that routes `/app1/` and `/app2/`.

---

## Step-by-Step Instructions

### Step 1: Start App 1 and App 2 in the background
```bash
python3 -m http.server 3000 --directory app1 &
python3 -m http.server 3001 --directory app2 &
```

### Step 2: Check both apps directly
```bash
curl http://localhost:3000
curl http://localhost:3001
```

### Step 3: Configure Nginx
```bash
# Copy the config
sudo cp myapps.conf /etc/nginx/sites-available/myapps

# Turn the site ON
sudo ln -s /etc/nginx/sites-available/myapps /etc/nginx/sites-enabled/

# Test syntax and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

### Step 4: Open both apps through Nginx (Port 80)
```bash
curl http://localhost/app1/
curl http://localhost/app2/
```

### Step 5: Clean up (stop background test servers)
```bash
pkill -f http.server
```
